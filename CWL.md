## Common Workflow Language (CWL) Overview

The Common Workflow Language (CWL) is an open, community-driven standard for defining portable and scalable workflows in data-intensive domains such as bioinformatics, genomics, and Earth Observation. The primary goal of CWL is to enable researchers and developers to share, reuse, and repurpose their workflows across different platforms and environments, thus fostering collaboration and accelerating scientific discovery.

In this section, we provide an overview of the key concepts and components of CWL, which will serve as the foundation for building EO applications.

### Key Concepts

CWL comprises two main components: CommandLineTools and Workflows. 

- **CommandLineTool**: A CommandLineTool is a description of a command-line program, its input parameters, output files, and any additional metadata. CommandLineTools are the building blocks of CWL workflows, encapsulating individual tasks and enabling their reuse across different workflows.

- **Workflow**: A Workflow is a sequence of CommandLineTools connected by input and output parameters. Workflows define the order of execution and data flow between the individual tools, allowing developers to model complex, multi-step analyses. 

### CWL Files and Syntax

CWL files are written in YAML or JSON formats, which makes them both human-readable and machine-parseable. The typical structure of a CWL file includes the following elements:

- **cwlVersion**: Specifies the version of the CWL standard used in the file (e.g., 'v1.0', 'v1.1', or 'v1.2').

- **class**: Indicates the type of CWL object being described, either 'CommandLineTool' or 'Workflow'.

- **inputs**: Defines the input parameters for the CommandLineTool or Workflow, including their types, default values, and any additional metadata.

- **outputs**: Describes the output files or values produced by the CommandLineTool or Workflow, including their types and how they are generated.

- **steps** (for Workflows only): Specifies the sequence of CommandLineTools in the Workflow, their input and output mappings, and any additional requirements or hints.

- **requirements** and **hints**: Lists any additional dependencies, resource requirements, or optional features needed to execute the CommandLineTool or Workflow.

### Portability and Scalability

One of the main advantages of using CWL is its portability and scalability. CWL workflows can be executed on various platforms, including local machines, high-performance computing clusters, and cloud environments, without any modification. This is made possible by the separation of concerns between workflow description and execution, where CWL focuses on the former and delegates the latter to CWL runners (also known as CWL engines or executors).

CWL runners are responsible for interpreting CWL files and managing the execution of CommandLineTools and Workflows on the target platform. Some popular CWL runners include:

- **cwltool**: The reference implementation of CWL, suitable for local execution and testing.
- other... 

By leveraging CWL runners, developers can build EO applications that are both portable and scalable, ensuring that their workflows can be easily shared, reused, and adapted to different environments and use cases.
