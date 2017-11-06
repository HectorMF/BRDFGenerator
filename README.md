# BRDF Generator

This is a simple program that generates Smith GGX BRDF lookup tables for the split sum approximation of the UE4-based PBR pipeline. 
These lookup tables are stored in KTX or DDS texture. The program generates two channel(RG) images using either 16 bit or 32 bit floating point precision. 

<p align="center"><img src="BRDF%20LUT.png?raw=true" /></p>

```
Note: It is important to use a floating point image to prevent clamping of values in your PBR pipeline.  
For most applications 16 bits is sufficient.  
```

Since this table is used for an approximation, some errors are introduced into the final renderings. The top image shows the reference path traced rendering, the middle shows a split sum approximation and the bottom shows the full approximation where N=V.  

<p align="center"><img src="BRDF%20Comparison.png?raw=true" /></p> 

When loading the texture into OpenGL, use the following filters to ensure that values are read correctly. 
```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
```

More Reading:

* http://gamedevs.org/uploads/real-shading-in-unreal-engine-4.pdf
* https://learnopengl.com/#!PBR/IBL/Specular-IBL


## Usage

```
BRDFGenerator -f [filename] -s [size] -n [samples] -b [bits]
```

Options:
```
-f [filename] The location and filename of the output image.  Must have an extension of ktx or dds.
-s [size] The size of the image in pixels per side. i.e. 512 will generate a 512x512 image. 
-n [samples] The number of BRDF samples per pixel. 
-b [bits]  The number of floating point bits used to store the data.  Can be either 16 or 32.
```

## Algorithm
```
For each pixel (x, y):
   Compute roughness (0 to 1.0) based on pixel x coordinate.
   Compute NoV (0 to 1.0) based on pixel y coordinate.
   Set view as float3(sqrt(1.0 - NoV * NoV), 0, NoV).
   Set normal as float3(0, 0, 1). 
   For each sample:
        Compute a Hammersely coordinate.
        Integrate number of importance samples for (roughness and NoV).
        Compute reflection vector L
        Compute NoL (normal dot light)
        Compute NoH (normal dot half)
        Compute VoH (view dot half)
        
        If NoL > 0
          Compute the geometry term for the BRDF given roughness squared, NoV, NoL
          Compute the visibility term given G, VoH, NoH, NoV, NoL
          Compute the fresnel term given VoH.
          Sum the result given fresnel, geoemtry, visibility.
   Average result over number of samples.
```

## Built With

The project has two header based dependencies, GLI and GLM, they are currently stored in the ext folder.

* [GLM](https://github.com/g-truc/glm) - Math Functions
* [GLI](https://github.com/g-truc/gli) - Image Saving

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
