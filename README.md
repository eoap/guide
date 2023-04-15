# Developer's Guide for Earth Observation Application Packages

## Introduction
This guide aims to help developers create Earth Observation (EO) applications using the Common Workflow Language (CWL). It provides an overview of the CWL, its key concepts, and demonstrates how to build a CWL-based EO application using practical examples 

[[more]](./INTRO.md)

## Common Workflow Language (CWL) Overview
CWL is a specification for describing analysis workflows and tools in a way that makes them portable and scalable across various platforms. CWL is designed to meet the needs of data-intensive science, such as genomics, bioinformatics, and Earth Observation  


### CWL Key Concepts

 * Workflow: A sequence of processing steps that describe how to transform input data into the desired output.
 * CommandLineTool: Represents a command-line tool in the CWL, including its inputs, outputs, and command-line options.
 * Workflow Step: A component of a workflow that represents a specific operation or task.
 * Requirements and Hints: Used to specify optional or mandatory configurations for a CWL tool or workflow.
 * Scatter and Gather: Techniques used to parallelize the execution of a workflow step over multiple input items.
 
[[more]](./CWL.md)


## Getting Started with CWL
 * Install a CWL runner, such as cwltool, which is a reference implementation of the CWL standard.
 * Familiarize yourself with the YAML syntax, as CWL documents are typically written in YAML.
 * 
[[more]](./START.md) [[alternative]](./START2.md)

 
## Building a CWL-based EO Application
### Define a CommandLineTool
Create a YAML file describing the command-line tool, including its inputs, outputs, base command, and requirements.
Use DockerRequirement to specify the Docker image to be used for running the tool.

### Create a Workflow
Define a Workflow class that orchestrates the CommandLineTools.
Specify the inputs, outputs, and processing steps in the workflow.

### Run the CWL Workflow

Use the CWL runner (cwltool) to execute the workflow, providing the workflow file and a YAML file containing input parameters.

## Advanced CWL Techniques
### Scatter and Gather
Use ScatterFeatureRequirement to parallelize the execution of a workflow step over multiple input items.
Select a scatter method: dotproduct, nested_crossproduct, or flat_crossproduct.

### Subworkflows
Create a reusable subworkflow by defining a Workflow class that can be referenced within another workflow.

### Additional Metadata

Provide authorship information, license details, and other relevant metadata using schema.org elements.

### Runtime Environment Resources

Specify the resource requirements for the application using the ResourceRequirement class, such as minimum and maximum RAM and CPU cores.

## Best Practices

 * Make use of existing CWL tools and workflows as a starting point.
 * Test your application with realistic input data and validate the outputs.
 * Document your application, including inputs, outputs, and processing steps.
 * Share your CWL-based EO application on public repositories, such as GitHub, to encourage collaboration and reuse.

With this guide, developers should be well-equipped to create Earth Observation applications using the Common Workflow Language, facilitating the development of portable, scalable, and reproducible EO applications.

