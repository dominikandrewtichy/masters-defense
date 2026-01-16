# Master's Thesis Defense - Speech Notes

## Title Slide
Good morning/afternoon. My name is Dominik Tichy, and I will be presenting my master's thesis titled "Lightweight Visualisation and Annotation of Volumetric and Segmentation Data in Mol*", supervised by Dr. Tomas Racek.

---

## Motivation

Microscopy data, particularly from techniques like Cryo-EM and tomography, is growing at an unprecedented rate. This data is crucial for structural biology research, enabling scientists to visualize complex biological structures.

To make this data accessible globally, web-based visualization tools have become essential. Among these, Mol* has established itself as the industry standard, serving as the primary viewer for major databases like RCSB PDB and AlphaFold DB.

To handle volumetric data specifically, the Mol* Volumes and Segmentations extension was developed. However, this extension has become technically obsolete following the introduction of the MolViewSpec standard.

The legacy client is difficult to maintain, isolated from the Mol* core, and has a rigid annotation system that lacks modern storytelling capabilities that researchers need for educational and publication purposes.

---

## Volumetric Data

Before diving into the goals, let me briefly explain the key concepts.

Volumetric data is essentially a 3D scalar field sampled at discrete points called voxels. Each voxel holds an intensity value - for example, electron density in X-ray crystallography or electrostatic potential in Cryo-EM.

This data is stored as a 3D array that gets mapped to physical coordinates, typically measured in Angstroms.

The main sources of volumetric data include X-ray crystallography, Cryo-EM, and various tomography techniques.

To visualize this data, we can use isosurface extraction which shows a shell at a specific threshold, direct volume rendering which shows the internal structure with transparency, or simple 2D slices for inspecting raw values.

---

## Segmentations

Segmentation is the process of converting raw continuous data into biologically meaningful discrete regions. For example, identifying where a mitochondrion begins and ends in a cellular tomogram.

There are three main representations:

First, lattice grids or voxel masks, where each voxel gets an integer ID. This preserves full spatial information but can be memory-intensive.

Second, meshes, which define surfaces using vertices and triangular faces. These are efficient for rendering and are typically derived from lattice data using algorithms like Marching Cubes.

Third, geometric primitives like spheres and cylinders. These are mathematically defined and very efficient for storage, though they only provide coarse approximations of biological shapes.

---

## Annotations

While segmentation provides the geometric boundaries, annotations assign semantic meaning.

Without annotations, a segmented mesh is just a generic shape. With annotations, it becomes an identified biological entity like "ATP Synthase Complex".

We distinguish two types:

Descriptive annotations provide semantic context - labels, descriptions, and links to external databases like Gene Ontology or PDB.

Visual annotations dictate how data is rendered - colors to distinguish components, opacity to reveal internal structures, and threshold values for volume rendering.

---

## Mol* VS Architecture

Now let me explain how Mol* Volumes and Segmentations works.

It has five major components:

The Preprocessor is the entry point - it converts input files to the OME-NGFF format and performs downsampling for multi-resolution streaming.

The Database stores the preprocessed data.

The Server handles HTTP requests and streams data to the browser.

The Client is a Mol* extension that renders the data and allows users to control visual parameters and interact with annotations.

Finally, the VSToolkit is a command-line tool for packaging data into CVSX archives - portable ZIP files containing all volumes, segmentations, and annotations needed to render a scene.

---

## Goals

The primary goal of this thesis is to design and implement a lightweight web application for visualizing and annotating volumetric and segmentation data in Mol*.

The specific objectives include:
1. Enabling upload, storage, and necessary conversion of volumetric and segmentation data files
2. Using modern Mol* features, specifically the MolViewSpec standard, for visualization
3. Implementing a REST API for programmatic access to datasets and annotations
4. Ensuring secure access through e-INFRA CZ authentication and shareable links
5. Deploying the solution using a CI/CD pipeline on the e-INFRA Kubernetes infrastructure

---

## Solution Overview

To achieve these objectives, I developed two complementary practical contributions.

First, a conversion library called cvsx2mvsx, which is a Python library published on PyPI. This library converts the legacy CVSX format used by Mol* VS to the modern MVSX and MVStory formats. It also provides a command-line interface for batch processing.

Second, a web application that provides a user-friendly interface for the conversion pipeline. It includes a WYSIWYG annotation editor, persistent storage with sharing capabilities, and a REST API for programmatic access.

---

## Conversion Library - Architecture

The library follows an ETL pipeline architecture - Extract, Transform, Load.

In the Extract phase, we validate and parse CVSX archives. Importantly, we use lazy loading to handle large datasets efficiently without loading everything into memory at once.

The Transform phase is the core of the library. Here we convert the data to an intermediate representation. This includes volume classification to determine whether data should be rendered as isosurfaces or slices, converting lattice segmentations to either volumes or meshes using the Marching Cubes algorithm, and handling geometric primitives through direct mapping or mesh generation.

Finally, in the Load phase, we serialize the intermediate representation to either MVSX or MVStory format.

Key features include memory-efficient lazy loading, automatic metadata enrichment from EMDB for contour levels, and automatic code generation for 4D datasets to create ready-made presentations in MolViewStories.

---

## Conversion Library - Data Mapping

This slide shows how each CVSX data type maps to MVS nodes.

For volumes, 3D data maps to volume nodes rendered as isosurfaces, while 2D slices - detected when any dimension equals 1 - map to grid slice representations.

Lattice segmentations have two options: they can be preserved as volume isosurfaces, or converted to meshes using the Marching Cubes algorithm. The mesh approach is useful for interoperability but increases file size.

Mesh segmentations pass through directly to MVS mesh nodes.

For geometric primitives, shapes natively supported by MVS - spheres, boxes, ellipsoids, and cylinders - map directly to the primitives node. Unsupported shapes like pyramids and tapered cylinders are procedurally converted to meshes.

Finally, all annotations - colors, labels, tooltips, opacity values - are mapped to the corresponding parameters on each MVS node.

---

## Conversion Library - Results

We verified the library on 14 diverse datasets from major repositories: EMDB, IDR, and EMPIAR.

Interestingly, our conversion actually improved upon the legacy client in several ways:
- We fixed missing volumes and restored color annotations that the legacy client failed to apply
- We corrected author-defined contour levels that were being ignored
- We implemented proper isosurface capping to create watertight meshes
- We batched geometric primitives, significantly improving performance for datasets with hundreds of shapes

File size overhead is minimal - less than 4% for most data types.

---

## Conversion Library - Architectural Impact

Beyond the immediate results, using MVSX instead of CVSX has significant architectural implications for the Mol* VS ecosystem.

First, it eliminates the need for custom loading logic in the Mol* VS Client. The MVSX format uses standard Mol* data loading through MolViewSpec, so there's no need for specialized volume and segmentation parsers anymore.

Second, this effectively obsoletes the CVSX format and therefore the VSToolkit component. The VSToolkit could still be useful, but repurposed to generate MVS output instead of the legacy format.

Third, the Mol* VS user interface no longer needs to be a heavyweight extension. It can now be implemented as a lightweight extension that works with the standard MVSJ tree structure, rather than being tightly coupled to custom data formats. This makes maintenance much easier and keeps it aligned with the evolving Mol* core.

---

## Web Application - Architecture

The web application uses a modern decoupled architecture.

The frontend is a React single-page application with integrated Mol* Viewer for 3D visualization.

The backend is built with FastAPI in Python, ensuring consistency with the conversion library. FastAPI provides native async support, which is critical for handling concurrent requests during file uploads and conversions.

For storage, we use PostgreSQL for relational data and MinIO for object storage to handle large binary files.

Key features include OpenID Connect authentication integrated with e-INFRA AAI, dual authentication supporting both JWT cookies for the web interface and API keys for programmatic access, asynchronous conversion that doesn't block the main thread, storage quota management, share links for collaboration, and the ability to export to MolViewStories for advanced annotation.

---

## Web Application - Deployment

The entire infrastructure is containerized with Docker and deployed on the CERIT-SC Kubernetes cluster.

We use Helm charts for configuration management, which allows declarative definition of all Kubernetes resources including deployments, secrets, persistent volumes, and ingress rules.

The release process is fully automated through a CI/CD pipeline implemented with GitHub Actions. Every push to main triggers a build, pushes images to the Harbor registry, and performs a rolling update on the cluster with zero downtime.

The application is available at the URLs shown - the web interface and the API documentation.

---

## Conclusion

To summarize, we achieved all the goals set out in the thesis:

We developed a conversion library, cvsx2mvsx, that bridges the gap between legacy and modern formats while actually improving upon the original visualization.

We implemented a web application with a graphical interface for conversion and annotation editing, deployed on Kubernetes with a full CI/CD pipeline.

Beyond the stated goals, our work corrected historical rendering artifacts in the legacy client, and the data mapping definitions we developed will serve as a foundation for the design of the new Mol* VS preprocessor.

Both the library and application are publicly available - the library on PyPI and the source code on GitHub.

Thank you for your attention. I'm happy to answer any questions.
