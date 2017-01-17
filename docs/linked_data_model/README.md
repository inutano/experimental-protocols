# The Experimental Protocols Linked Data Model

The model to represent experimental protocols as a workflow.

## The basic structure of the protocol description

A protocol as a workflow consists of one or more processes (procedures) connected by the predicate which connects processes each other in order of the execution. A process should have a label.

Programs which produce the data in this linked data model must assign URIs for the protocol and each process. Though there is no strict rules for designing URIs, but here we use <https://mylab.org/resources/RNAseq> for the URI of the protocol, and URIs with indexes connceted by dots as URIs of the processes. Multiple dots indicate the depth of the nested processes.

Examples below are using fake ontology terms with prefix "proto", which currently not yet decided from which ontology we employ.

The protocol is from the article of the nature protocols, [Defining transcribed regions using RNA-seq](http://www.nature.com/nprot/journal/v5/n2/full/nprot.2009.229.html).

```ttl
@prefix proto: <https://ontologies-to-be-defined.org/resources/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix mylab: <https://mylab.org/resources/> .

# RNA-seq protocol is an instance of Protocol class

mylab:RNAseq a proto:Protocol ;
  rdfs:label "Defining transcribed regions using RNA-seq" .

# RNA-seq protocol has three processes

mylab:RNAseq proto:has_process mylab:RNAseq.1, mylab:RNAseq.2, mylab:RNAseq.3 .

# Define three processes

mylab:RNAseq.1 a proto:Process ;
  rdfs:label "Yeast culture" .

mylab:RNAseq.2 a proto:Process ;
  rdfs:label "RNA extraction" .

mylab:RNAseq.3 a proto:Process ;
  rdfs:label "RNA purification" .

# Order of the execution

mylab:RNAseq.1 proto:is_followed_by mylab:RNAseq.2 .
mylab:RNAseq.2 proto:is_followed_by mylab:RNAseq.3 .
```

A process can have child processes (sub processes).

```ttl
# Yeast culture process has three steps

mylab:RNAseq.1 proto:has_process mylab:RNAseq1.1, mylab:RNAseq1.2, mylab:RNAseq1.3 .

mylab:RNAseq1.1 a proto:Process ;
  rdfs:label "Yeast culture step 1" .

mylab:RNAseq1.2 a proto:Process ;
  rdfs:label "Yeast culture step 2" .

mylab:RNAseq1.3 a proto:Process ;
  rdfs:label "Yeast culture step 3" .

mylab:RNAseq1.1 proto:is_followed_by mylab:RNAseq1.2 .
mylab:RNAseq1.2 proto:is_followed_by mylab:RNAseq1.3 .
```

A child can have children as well (nested processes).

```ttl
# Yeast culture step 1

mylab:RNAseq1.1 proto:has_process mylab:RNAseq1.1.1 .

mylab:RNAseq1.1.1 a proto:Process ;
  rdfs:label "Preculture of S. pombe" .

# Yeast culture step 2

mylab:RNAseq1.2 proto:has_process mylab:RNAseq1.2.1, mylab:RNAseq1.2.2 .

mylab:RNAseq1.2.1 a proto:Process ;
  rdfs:label "Transfer precultured yeast to fresh YES media" .

mylab:RNAseq1.2.2 a proto:Process ;
  rdfs:label "Incubation" .

# Yeast culture step 3

mylab:RNAseq1.3 proto:has_process mylab:RNAseq1.3.1, mylab:RNAseq1.3.2 .

mylab:RNAseq1.3.1 a proto:Process ;
  rdfs:label "Separate the culture" .

mylab:RNAseq1.3.2 a proto:Process ;
  rdfs:label "Centrifuge" .
```

Note that the Protocol class is a special class inherited from the Process class, which means a Protocol can have a Protocol as a child, or a Process can have a Protocol as well.

```ttl
mylab:MultiOmics a proto:Protocol ;
  proto:has_process mylab:Exome, mylab:RNAseq, mylab:ChIPseq .
```

## The structure of the process description

A process must have input and output. A process can have multiple inputs/outputs. If a process have children, then the input of the first child or the output of the last child would be the input/output of the parent.

```
mylab:RNAseq1.1.1 a proto:Process ;
  rdfs:label "Preculture of S. pombe" ;
  proto:has_input mylab:RNAseq0-1, mylab:RNAseq0-2 ;
  proto:has_output mylab:RNAseq1.1.1-1 .

mylab:RNAseq0-1 rdfs:label "yeast cells picked from a YES agar plate" .
mylab:RNAseq0-2 rdfs:label "YES media" .

mylab:RNAseq1.1.1-1 rdfs:label "precultured yeast cells" .
```

## The typed process

(to be written)

## The syntax of conditionals and loops

(to be written)
