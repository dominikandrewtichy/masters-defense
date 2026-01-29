Good morning. My name is Dominik Tichý and today I would like to present to you the results of my master's thesis work. The topic of my thesis is "Lightweight Visualisation and Annotation of Volumetric and Segmentation Data in Molstar".

---

For the past 10 years or so we have been living through a so called "Resolution Revolution". This has been possible thanks to new advancements in microscopy techniques, especially in electron microscopy, which have accelarated the growth of available microscopy data. These microscopy data are very important in many areas of research such as structural biology where it is used for example in drug design. A perfect example of this is the fast development of the covid vaccines which was possible thanks to microscopy data of the covid's spike protein.

---

So what are these microscopy data exactly? They consist of primarily of volumes which are the raw measurements from microscopes. Essentially they are just 3d grids with continuous values that represent some density values. When these 3d grids are visualized we often use a so called isovalue which is a threshold used to determine is the density values should be included or excluded in the volume visualization. Take for example these 3 images. The one on the left has a very low isovalue therefore the resulting 3d surface is very noisy while the middle and right image have a higher isovalue and therefore they have less noise and look more closely like the actual object that was imaged.

---

As we have seen on the previous slide the volumes are just a gray glob that doesn't tell us much about the biological meaning of the image. Volume segmentations do just that. They split the 3d image into individual parts so that we can have a better understanding of the positoin of various biological objects within the image. In this thesis, we have worked with 3 types of segmentations. Lattice segmentations are essentially just binary maps that tell us which values in the volumes 3d grid correspond to a biological object. Meshes on the other hand are just a set of vertices and triangles that define the surrounding surface of object. And finally geometric primitives are basic shapes like spheres or cylinders that are defined by mathematical values such as radius and center for sphere. Unlike lattices and meshes they don't give us a precise volumetric shape or outline shape but instead they just give us an approximation of the shape and an approximation of the position.

---

That being said, these volumes and segmentations are useless if researchers can’t easily visualize them, annotate them, and share them with the broader research community.
For this we have many 3d web visualizers. Currently the defacto standard in structural biology is the Molstar viewer which allows to visualize various biomolecular data like proteins or macromolecular complexes.
Apart from this there exists an extension for Molstar called Molstar Volumes and Segmentations or molstar vs for short which provides support for visualizing volumetric and segmentation data. This extension was a good proof of concept, however, it has since become an unmaintained project which lacks behind the rest of the molstar ecosystem and is therefore mostly unusable.

Naturally, these limitations raised the need to update the extension in order to continue support for volumes and segmentations in Molstar. Fortunately, there have since been other advancements in the Molstar ecosystem, most notably the introduction of MolViewSpec. MolViewSpec is a declarative standard which allows to describe molecular scenes in a tree-like structure. This is great because it allows us to describe exactly what we want the scene to look like.
For example, we can use it to let's say show this protein, zoom in on this part of it, color it, set this opacity etc you get the idea... 
Before this we would have to specify all of this using custom Javascript wrappers around Molstar which would be very difficult. Now MolViewSpec streamlines this all into a single JSON file that we can load into any up to date molstar viewer instance.

---

So this seemed like a very good alternative to use for the molstar VS replacement. Now we just 

---



To summarize my work, I have firstly replaced the Molstar Volume and Segmentation client with MolViewSpec which will server as a blueprint for the next generation of the Molstar Volseg preprocessor. Secondly, I have created a conversion library for migrating data from the old CVSX file format to the new MolViewSpec file formats. Thirdly, I have built a web application to provide access to this tool to researchers and to visualize and share the results. Finally, this work has contributed to a research paper which was published in a scientific journal.
Ok that would be it from me, thank you for your attention.