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

```ttl
mylab:RNAseq1.1.1 a proto:Process ;
  rdfs:label "Preculture of S. pombe" ;
  proto:has_input mylab:RNAseq0-1, mylab:RNAseq0-2 ;
  proto:has_output mylab:RNAseq1.1.1-1 .

mylab:RNAseq0-1 rdfs:label "yeast cells picked from a YES agar plate" .
mylab:RNAseq0-2 rdfs:label "YES media" .

mylab:RNAseq1.1.1-1 rdfs:label "precultured yeast cells" .
```

## Conditionals and loops

Experimental protocols written in natural language often have terms to specify special operations like jumps, switches, repeats, or loops to describe complicated workflow structures simply. In this linked data model, however, we aim to achieve to describe those special relations between processes without introducing special concept as much as possible, even if it seemed to be redundant.

### The concept of condition

In this data model, a concept "Condition" is used to evaluate the next process to be processed. Even if the simple straightforward workflow with two processes have condition.

```ttl
# two processes
mylab:RNAseq1 a proto:Process .
mylab:RNAseq2 a proto:Process .

# process 1 is followed by process 2
mylab:RNASeq1 proto:is_followed_by mylab:RNAseq2 .

# process 1 has a condition, and the condition has a link to the next process to be processed if the condition was fulfilled
mylab:RNASeq1 proto:has_condition mylab:RNAseq1Cond ;
mylab:RNAseq1Cond proto:process mylab:RNAseq2 .

# In this case, condition is just an evaluation of exit status.
mylab:RNASeq1Cond rdfs:label "Is mylab:RNASeq1 finished correctly?"
```

Usually, the second process should be executed if the previous one is processed correctly. So the condition can be omitted, and processes are simply described as follows.

```ttl
# two processes and a link between them
mylab:RNAseq1 a proto:Process .
mylab:RNAseq2 a proto:Process .
mylab:RNASeq1 proto:is_followed_by mylab:RNAseq2 .
```

### Conditional switches

Some experimental procotols have a process branch. Usually, an output of the process is evaluated to decide which process should be processd at next. Let's say there are process2 and process3 and either of them will be executed after process1, evaluating output of process1.

```ttl
# three processes
mylab:RNAseq1 a proto:Process .
mylab:RNAseq2 a proto:Process .
mylab:RNAseq3 a proto:Process .

# process1 switches to process2 or process3, which means process1 is followed by both
mylab:RNAseq1 proto:is_followed_by mylab:RNAseq2, mylab:RNAseq3 .

# process1 has two conditions linked to process2 and process3
mylab:RNAseq1 proto:has_condition mylab:RNAseq1cond1, mylab:RNAseq1cond2 .
mylab:RNAseq1cond1 proto:process mylab:RNAseq2 .
mylab:RNAseq1cond2 proto:process mylab:RNAseq3 .

# two conditions evaluate output of process1
mylab:RNAseq1 proto:has_output mylab:RNAseq1-1 .
mylab:RNAseq1cond2 proto:evaluate mylab:RNAseq1-1 .
mylab:RNAseq1cond3 proto:evaluate mylab:RNAseq1-1 .
```

### Conditional loops

A statement like "repeat until the output volume > 10ml" is frequently described in experimental protocols. This can be represented as an alternative of switching statement, a process has itself as the next process.

```ttl
# repeat process1 and the next process2
mylab:RNAseq1 a proto:Process .
mylab:RNAseq2 a proto:Process .

# process1 has process1 itself as the next process, having process2 as well
mylab:RNAseq1 proto:is_followed_by mylab:RNAseq1, mylab:RNAseq2 .

# process1 has two conditions likned to process1 itself and process2
mylab:RNAseq1 proto:has_condition mylab:RNAseq1cond1, mylab:RNAseq1cond2 .
mylab:RNAseq1cond1 proto:process mylab:RNAseq1 .
mylab:RNAseq1cond2 proto:process mylab:RNAseq2 .
```

### N times loop

Statements of repeat with frequency like "repeat 3 times" should be represented by the simple repeat of the same processes. Repeated processes may have the same procedure, but have different input and output in terms of the physical state like volume or concentration, so they should be described as the different processes.

It seems to be redundant, and the graphical representation of the protocols may have to handle this, but this is data.

```ttl
# Repeat process1 3 times: three identical procedures, but have different input/output
mylab:RNAseq1.1 a proto:Process ;
  rdfs:label "Process 1 Repeat 1" ;
  proto:has_input mylab:RNAseq1.0-1 ;
  proto:has_output mylab:RNAseq1.1-1 .

mylab:RNAseq1.2 a proto:Process ;
  rdfs:label "Process 1 Repeat 2" ;
  proto:has_input mylab:RNAseq1.1-1 ;
  proto:has_output mylab:RNAseq1.2-1 .

mylab:RNAseq1.3 a proto:Process ;
  rdfs:label "Process 1 Repeat 1" ;
  proto:has_input mylab:RNAseq1.2-1 ;
  proto:has_output mylab:RNAseq1.3-1 .
```

## The typed process

(to be written)
