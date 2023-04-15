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

### Workflow
A workflow is a sequence of processing steps that describe how to transform input data into the desired output. In CWL, a workflow is represented by a YAML file that describes the inputs, outputs, and steps of the workflow.

Here's an example of a simple CWL workflow that takes a text file as input, converts it to uppercase, and saves the result to a new file:

```yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_file:
    type: File
    inputBinding:
      position: 1
outputs:
  output_file:
    type: File
    outputBinding:
      glob: $(inputs.input_file.basename).upper.txt
steps:
  uppercase:
    run: uppercase.cwl
    in:
      input_file: input_file
    out:
      output_file: output_file
```

This workflow has one input, input_file, which is a text file, and one output, output_file, which is also a text file. The workflow has one step, uppercase, which runs the uppercase.cwl CommandLineTool.

### CommandLineTool
A CommandLineTool represents a command-line tool in CWL, including its inputs, outputs, and command-line options. A CommandLineTool is defined in a separate YAML file and can be reused across multiple workflows.

Here's an example of a simple CWL CommandLineTool that takes a text file as input, converts it to uppercase, and saves the result to a new file:

```yaml
cwlVersion: v1.0
class: CommandLineTool
baseCommand: tr
inputs:
  input_file:
    type: File
    inputBinding:
      position: 1
  output_file:
    type: File
    outputBinding:
      glob: $(inputs.input_file.basename).upper.txt
arguments:
  - valueFrom: "'[:upper:]'"
stdout: $(inputs.output_file.path)
```

This CommandLineTool uses the tr command to convert the input file to uppercase. The input_file input specifies the input file, and the output_file output specifies the output file. The arguments field specifies the command-line options to pass to tr.

### Workflow Step
A workflow step is a component of a workflow that represents a specific operation or task. A workflow step can reference a CommandLineTool, as we saw in the previous example, or it can define its own operation.

Here's an example of a simple CWL workflow step that takes a text file as input, converts it to uppercase, and saves the result to a new file:

```yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_file:
    type: File
    inputBinding:
      position: 1
outputs:
  output_file:
    type: File
    outputBinding:
      glob: $(inputs.input_file.basename).upper.txt
steps:
  uppercase:
    in:
      input_file: input_file
    out:
      output_file: output_file
    run:
      class: CommandLineTool
      baseCommand: tr
      inputs:
        input_file:
          type: File
          inputBinding:
            position: 1
        output_file:
          type: File
          outputBinding:
            glob:
```

Requirements and Hints
Requirements and Hints are used to specify optional or mandatory configurations for a CWL tool or workflow. Requirements specify things like the minimum amount of memory or disk space required for a tool to run, while Hints provide additional information about how the tool should be executed.

Here's an example of how to specify Requirements and Hints for a CWL tool:


```yaml
cwlVersion: v1.0
class: CommandLineTool
baseCommand: my_tool.py
inputs:
  input_file:
    type: File
    inputBinding:
      position: 1
requirements:
  - class: ResourceRequirement
    ramMin: 4G
    coresMin: 2
hints:
  - class: DockerRequirement
    dockerImageId: my_tool:v1
```

In this example, the ResourceRequirement specifies that the tool requires at least 4GB of RAM and 2 CPU cores to run. The DockerRequirement specifies that the tool should be executed in a Docker container with the image my_tool:v1.

### Scatter and Gather
Scatter and Gather are techniques used to parallelize the execution of a workflow step over multiple input items. Scatter is used to split a list of input items into multiple chunks, which can be executed in parallel, and Gather is used to combine the output of those parallel executions into a single output.

Here's an example of how to use Scatter and Gather in a CWL workflow:

```yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_files:
    type: File[]
    inputBinding:
      position: 1
steps:
  uppercase:
    in:
      input_file: input_files
    out:
      output_files: output_files
    scatter: input_file
    run:
      class: CommandLineTool
      baseCommand: tr
      inputs:
        input_file:
          type: File
          inputBinding:
            position: 1
        output_file:
          type: File
          outputBinding:
            glob: $(inputs.input_file.basename).upper.txt
    scatterMethod: dotproduct
    gather: output_files
    outputEval: |
      outputs:
        output_files:
          type: File[]
          outputBinding:
            glob: "*.upper.txt"
            loadContents: true
```

In this example, the workflow takes a list of input files as input and uses Scatter to split the list into multiple chunks. The uppercase workflow step is executed in parallel for each chunk of input files, and the output is combined using Gather. Finally, the output files are collected and saved to a list called output_files. The scatterMethod specifies how the input list should be split, and the outputEval field specifies how the output should be combined.
