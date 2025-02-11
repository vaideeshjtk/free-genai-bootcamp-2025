## Business Goal

As a Solution Architect after consulting with real-world AI Engineers, you have been tasked to create architectural diagram(s) that serve as a teaching aid to help stakeholders understand their key components of GenAI workloads. The outcome is to help let stakeholders visualize possible technical paths, technical uncertainty when adopting GenAI.

We are guiding key stakeholders through the technical landscape without directly prescribing solutions while fostering informed discussions about infrastructure choices, integration patterns, and system dependencies across the organization.

We can use all levels of technical diagramming to achieve our goal

- Diagram to Stakeholders - Conceptual
- Diagram of Sentence Constructor GenAI - Conceptual

## Levels of Diagram 
• Conceptual • Logical • Physical
These address the different types of Stakeholders to whom we are going to talk 

# Conceptual
E.g., Conceptual is a 10,000ft overview these are things that you are going to solve.
How things feed to other one, how things lay out with one another... The diagram that I have attached here with the file name Free GenAI Bootcamp Conceptual Diagram contains a conceptual diagram.

# Logical
This in for Blueprint. It has Names, Measurements, and how things are laid out, This is for a Project Manager or Solutions Architects etc.

# Physical
This is the actual list of materials. Eg How many bricks are needed, and how many servers are needed? It 
 includes the name of Service to the IP Address to Ports. How a physical diagram looks is like This server is hosted in AWS in US East 1c with so and so configurations and specs. This is for Engineers.

## Business Requirements

We are going to build an application that will potentially replace or assist the teachers in order to help students of the language learning school.

## Functional Requirements

The company wants to invest in owning its infrastructure.
The reason is that they there is a concern about the privacy of user data and also a concern that the cost of managed services for GenAI will greatly rise in cost.
They want to invest an AI PC where they can afford to spend of 10-15K,
They have 300 active students, and students are located within the city of Montevideo.

## Non-Functional Requirements
- Performance - OKish
- Scalability - No issue
- Security - Important to protect user data
- Usability - Very important to make students stay attracted in the activities and portal.

## Tooling 
GenAI

## Risk

- To protect the student's and teacher's PII data and send via Secure API
- The GenAI must not give any misleading comments or response to students - make it with guardrails
## Assumptions

We are assuming that the Open-source LLMs that we choose will be powerful enough to run on hardware with an investment of 10-15K.


We're just going to hook up a single serve in our office to the internet and we should have enough bandwidth to serve the 300 students.

## Data Strategy

There is a concerned of copyrighted materials, so we must purchase and supply materials and store them for access in our database.
Core 2000 words

## Considerations

- Input - Output: Text - Text, Text - Audio, Audio - Text, Audio - Audio
- Model - DeepSeek R1
We are considering Deepseek's R1 to be run locally with Ollama.


We're considering using IBM Granite because its a truley open-source model with training data that is traceable so we can avoid any copyright issues and we are able to know what is going on in the model.

https://huggingface.co/ibm-granite