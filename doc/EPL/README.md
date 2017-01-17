# How to describe experimental protocol in EPL format

**This document is not yet done. Please contact inutano@gmail.com for more details.**

## What is this

This is a guideline for biologists to translate the experimental protocols written in natural language to the Experimental Protocol Language (EPL) format.

## How it works

To represent information using the semantic web technology such as URI, ontologies, or RDF format  is very powerful and has a lot of benefits. We have been working to develop a framework based on the technology to describe protocols of biological experiments in a machine readable format, but translating protocols directly to the RDF format using URI or controlled vocabulary requires skills and expertise of the semantic web. Here, we introduce the way for biologists, who are not familiar with semantic web, to convert protocols in natural language to the structured document in EPL format using markdown syntax. Protocols described in this syntax will be easily serialised to RDF by the converter, which we are going to develop. **The syntax and terms are not fully fixed, and still has lots of room to improve.** This document will be updated as well. Any reports or comments are welcome.

## Basic syntax

Let's start with a very simple example taken from the article [Defining transcribed regions using RNA-seq](http://www.nature.com/nprot/journal/v5/n2/full/nprot.2009.229.html). This article is published on [Nature protcols](http://www.nature.com/nprot/), and is already fine structured (See "procedure" section). It is recommended to follow this style when you manage your own protocol.

Here is the yeast culture part of the protocol:

> Yeast culture  
> 1. Start preculture of S. pombe growing at 32 °C in 50 ml flask overnight by inoculating 30 ml of YES media with cells picked from a YES agar plate. For timing estimates of the different steps, see Figure 1.  
> 2. From the exponentially growing preculture (OD 595 <0.5), transfer a sufficient volume of precultured yeast to 100 ml of fresh YES media such that the final OD of the harvested material will be between 0.2 and 0.5 (i.e., inoculate 100 ml YES with a small amount of exponential-phase preculture and incubate at 32 °C until it reaches OD 595 of 0.2–0.5).  
> 3. Separate the culture into two 50 ml Falcon tubes and centrifuge for 2 min at 1,500g at 25 °C to form cell pellets.

We are going to translate this into the structured document by using markdown format. If you are not familiar with markdown, take a look at [Mastering Markdown](https://guides.github.com/features/mastering-markdown/). If you do not have your favourite editor, [Atom Editor](https://atom.io) is worth a try. It is a fine text editor to write and preview markdown document.

So, the protocol above has a title "Defining transcribed regions using RNA-seq" and the first section title "Yeast culture". The section "Yeast culture" has three steps. First, we describe them as headers of the document.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

### 2

### 3
```

Next, translate sentences to processes. In EPL, a protocol is represented by a set of processes. The very first process in the section 1 is written in the first sentence as *"preculture of S. pombe"*. So this is going to be a label of the first process. The processes in a step are described as list items of markdown syntax, using the keyword **process** and a colon to separate the keyword and the value related to the keyword, like `keyword: value`. The keyword **process** should have a label of the process as its value. A string value like label must be double-quoted. So, the first process goes like this:

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"

### 2

### 3
```

A process has to have its **input** and **output**. For **input**, the original protocol says we will require *cells picked from a YES agar plate*. There should be a process to pick cells from a plate, but this time it is not described in this protocol, so we just write it on a label. The **input** value also has a label, so put some words on it. I would use "cells picked from a YES agar plate" from the text, and add a word "yeast" just to make sure it is. It depends on you how explicit  to write labels, but at least they should have something to distinguish from each other.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"

### 2

### 3
```

To start preculture, we also need the media and the container. The protocol says it takes *30 ml of YES media* and *50 ml flask*. Let's consider how we can describe them.

For *30 ml of YES media*, the item *YES media* is one of the ingredients that are going to be precultured by the preculture process. So, this should be an **input** as well as the yeast cells. Besides, *50 ml flask* is the container which is essential to the preculture process itself, like scissors for cutting process. So we use **container** keyword for the preculture process to show this is an attribute for **process**.

> What should be **input** and what should not depend on the interpretation of the processes, so this rule may be altered in the future.

So, items for the first process are described as follows.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"

### 2

### 3
```

Now, it is clear that three items are required to start the process. Yet the culture condition is unclear, so let's put the growing conditions *growing at 32 °C* and *overnight* into it. The conditions are attributes related to the process itself, so put them in the same level as **container**.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"

### 2

### 3
```

Well, It seems that the information in the first sentence are all encoded in a structured form! Keywords like "container", "temperature", or "time" must be strictly controlled for sharing, but leave it for the time being (we are working on it). The sentence "For timing estimates.." looks unnecessary, so we're all done?

Wait. Before moving to the next, we need to describe what is produced by the first process so that the next process can specify it as an input. Use an **output** keyword and label it as you like. This time, the process will produce only one output, precultured yeast cells, but remember a process can produce multiple outputs as well as it can have multiple inputs.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"

### 2

### 3
```

The RDF converter, which we are going to develop, will parse this format and convert to the RDF format. We need to make sure that the description in EPL format has enough information to be converted (imagine you are a robot which cannot "read between the lines"), but we can skip to represent information which the converter can complement. For example, the product of the first process described by **output** keyword is "precultured yeast cells", and no other information is provided, but the converter knows that it is yeast cells picked from a plate inside the 50 ml flask with 30 ml YES media by interpreting the process.

OK, let's move on. The second sentence looks being able to be separated to two processes, *transfer* and *incubate*. So put two processes under the second step.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"

### 2

- process: "Transfer precultured yeast to fresh YES media"
- process: "Incubation"

### 3
```

Again, we are going to put the information under the processes. First, the transfer process will have an input, but we need to make sure it is identical to the output of the previous process. How?

Here's a magic to specify **input**. We have a implicit rule to assign IDs to processes and their outputs. To specify the output from the very first process on this protocol, "precultured yeast cells", use "1.1-1". The first "1" of "1.1-1" indicates "Step 1", represented by `### 1`. The second "1" indicates "the first process of the step". Connecting them with a dot to represent the level of the process. The final "-1" indicates "the first output of the process".

By using this rule, we can identify all the outputs from all the processes. Use a label for **input** if it is not produced from the previous processes, otherwise use ID. At the same time, using ID for **input** means using the label of the **output** as that of the **input**, so don't forget put a label to the **output**.

So, the document goes like this:

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe" <!-- This process is "1.1" -->
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells" <!-- This output is "1.1-1" -->

### 2

- process: "Transfer precultured yeast to fresh YES media" <!-- This process is "2.1"-->
  - input: "1.1-1"
  - output: "precultured yeast in fresh YES media" <!-- This output is "2.1-1" -->
- process: "Incubation"
  - input: "2.1-1" <!--- This specifies "precultured yeast in fresh YES media" --->
  - output: "yeast culture" <!-- This output is "2.2-1" -->

### 3
```

According to the first phrase, *From the exponentially growing preculture (OD 595 <0.5)*, the **input** for the process "2.1" has a specification for its density. We may need to have another process to adjust the density of the yeast culture to this condition before transferring the yeast culture, but for this time, put this information to the **output** of the previous process to let the process "1.1" to produce the ideal output for the next process, since this process is the only one process which uses the precultured yeast.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"
    - density: "OD 595 <0.5" <!-- Specifies density of the output -->

### 2

- process: "Transfer precultured yeast to fresh YES media"
  - input: "1.1-1"
  - output: "precultured yeast in fresh YES media"
- process: "Incubation"
  - input: "2.1-1"
  - output: "yeast culture"

### 3
```

Put the other information as well.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"
    - density: "OD 595 <0.5"

### 2

- process: "Transfer precultured yeast to fresh YES media"
  - input: "1.1-1"
    - volume: "a sufficient volume"
  - input: "fresh YES media"
    - volume: "100 ml"
  - output: "precultured yeast in fresh YES media"
- process: "Incubation"
  - input: "2.1-1"
  - output: "yeast culture"
    - density: "OD 595 of 0.2–0.5"
  - temperature: "32 °C"

### 3
```

The incubation process has to have the termination condition, such as time. For this incubation process, **output** has the specific density (Though it is not 'specific', anyway.), so we expect it is interpreted as the process ends if the density reaches the condition.

Let's move to the third step. It will be like this:

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"
    - density: "OD 595 <0.5"

### 2

- process: "Transfer precultured yeast to fresh YES media"
  - input: "1.1-1"
    - volume: "a sufficient volume"
  - input: "fresh YES media"
    - volume: "100 ml"
  - output: "precultured yeast in fresh YES media"
- process: "Incubation"
  - input: "2.1-1"
  - output: "yeast culture"
    - density: "OD 595 of 0.2–0.5"
  - temperature: "32 °C"

### 3

- process: "Separate the culture"
  - input: "2.2-1"
  - container: "Falcon tube"
    - size: "50 ml"
  - container: "Falcon tube"
    - size: "50 ml"
  - output: "Separated culture"
  - output: "Separated culture"
- process: "Centrifuge"
  - input: "3.1-1"
  - input: "3.1-2"
  - output: "Pellet and supernatant"
  - output: "Pellet and supernatant"
  - time: "2 min"
  - temperature: "25 °C"
  - gravity: "1,500g"
```

The process "Separate the culture" produces two Falcon tubes filled with yeast culture. So, it has one **input**, two **container**, and two **output**. The containers are same thing, so are outputs. It is not painful to write two same lines, but it is going to be ridiculous to write hundreds of same lines. So we describe the number of same items with the **#** keyword.

```markdown
# Defining transcribed regions using RNA-seq

## Yeast culture

### 1

- process: "Preculture of S. pombe"
  - input: "yeast cells picked from a YES agar plate"
  - input: "YES media"
    - volume: "30 ml"
  - container: "flask"
    - size: "50 ml"
  - temperature: "32 °C"
  - time: "overnight"
  - output: "precultured yeast cells"
    - density: "OD 595 <0.5"

### 2

- process: "Transfer precultured yeast to fresh YES media"
  - input: "1.1-1"
    - volume: "a sufficient volume"
  - input: "fresh YES media"
    - volume: "100 ml"
  - output: "precultured yeast in fresh YES media"
- process: "Incubation"
  - input: "2.1-1"
  - output: "yeast culture"
    - density: "OD 595 of 0.2–0.5"
  - temperature: "32 °C"

### 3

- process: "Separate the culture"
  - input: "2.2-1"
  - container: "Falcon tube"
    - size: "50 ml"
    - #: 2
  - output: "Separated culture"
    - #: 2
- process: "Centrifuge"
  - input: "3.1-1"
    - #: 2
  - output: "Pellet and supernatant"
    - #: 2
  - time: "2 min"
  - temperature: "25 °C"
  - gravity: "1,500g"
```

Now it looks better.

There will be a trick to pass the multiple same items to the following processes. It is going to be explained in the next section of this document.

## Special syntax

Sometimes we cannot describe the whole protocol with a simple chain of the processes. We provides some special syntax rules to describe more complex structure of experimental protocols.

### Filter syntax

Filter syntax is a wrapper for the **process** to provide control flow syntax in EPL.

#### Loop filter

```markdown
- filter: "Vortex sample every 10 mins in 1 hour incubation"
  - loop: 6
    - process: "Place 10 min"
      - input: "5.2-1"
      - output: "yeast cells in acidic phenol–chloroform"
      - temperature: "65 °C"
      - time: "10 min"
    - process: "Vortex"
      - input: "7.1.1-1"
      - output: "yeast cells in acidic phenol–chloroform"
      - time: "10 s"
```

#### Case filter

```markdown
- filter: "Incubation time"
  - case: "user decision"
  - when: "Incubate 1 min"
    - process: "Incubate 1 min"
      - input: "22.1-1"
      - temperature: "65 °C"
      - time: "1 min"
      - output: "sample in an Eppendorf tube"
  - when: "Incubate 10 min"
    - process: "Incubate 10 min"
      - input: "22.1-1"
      - temperature: "RT"
      - time: "10 min"
      - output: "sample in an Eppendorf tube"
```

#### Each filter

```markdown
- filter:
  - each: "Iterate following steps for each tube"
    - input: "3.2-1"
      - #: 2
```

### Nested processes

```markdown
- process: "PCR"
  - process: "heating"
    - temperature: "98 °C"
    - time: "30 s"
    - input: "40.1-1"
    - output: "heated PCR mix"
  - filter: "18 cycles"
    - loop: 18
      - process: "denaturation"
        - temperature: "98 °C"
        - time: "10 s"
        - input: "40.2.1-1"
        - output: "PCR mix"
      - process: "annealing"
        - temperature: "65 °C"
        - time: "30 s"
        - input: "40.2.2.1-1"
        - output: "PCR mix"
      - process: "extension"
        - temperature: "72 °C"
        - time: "30 s"
        - input: "40.2.2.2-1"
        - output: "PCR mix"
  - process: "cooling"
    - temperature: "72 °C"
    - time: "300 s"
    - input: "40.2.2-1"
    - output: "PCR mix"
  - process: "keep"
    - temperature: "4 °C"
    - expiration_time: "Infinite"
    - input: "40.2.3-1"
    - output: "amplified cDNA fragment"
```

### Skippable process

```markdown
- process: "Store"
  - temperature: "4 °C"
  - time: "overnight"
  - input: "40.2-1"
  - output: "amplified cDNA fragment"
  - skippable: true
```

*This document finishes here, will be updated soon!*
