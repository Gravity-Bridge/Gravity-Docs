# Gravity Bridge Software Architecture

## **Gravity Bridge Context diagram**

**Gravity Bridge Context** diagram shows how the Gravity Bridge fits into the world around it.

The Gravity Bridge software system is presented as a box in the centre, surrounded by its users and the  external system ( Ethereum ) that it interacts with. It allows us to step back and see the big picture.

The focus is on people, actors, roles and software systems rather than technologies, protocols and other low-level details.

Intended audience: Everybody, both technical and non-technical people, inside and outside of the software development team.

![Gravity Bridge Context Diagram](../assets/Gravity-Bridge-Context-Diagram.svg)

## **Gravity Bridge Container diagram**

Zoom-in to the Gravity Bridge System boundary with a **“Container” diagram**.
Container is a separately runnable/deployable unit presented with the light blue boxes. A container executes code or stores data (not a docker container).

Gravity Bridge Container diagram shows the high-level shape of the Gravity Bridge software architecture and how responsibilities are distributed across it. Diagram also shows the major technology choices and how the containers communicate with one another. It's a simple, high-level technology focussed diagram that is useful for software developers and support/operations staff alike.
Description for each box should be appropriate to the level of detail that the diagram shows. Relations/arrows should be just single directed with some general description or main intention.

This diagram can help developers to understand what processes Gravity Bridge uses, what ports are open, what protocols are used, deployment procedure goal etc.

![Gravity Bridge Container Diagram](../assets/Gravity-Bridge-Container-Diagram.svg)

## **Orchestrator - Component diagram**

**Orchestrator - Component diagram** decomposes Orchestrator container (light blue box) further to identify the major structural building blocks and their interactions. It shows how the Orchestrator is made up of a number of components, what each of those components are, their responsibilities and the technology/implementation details. todo: fill in the details

Supporting elements: Containers (within the software system in scope) plus people and software systems directly connected to the components.

Intended audience: Software architects and developers.

![Orchestrator - Component Diagram](../assets/Orchestrator-Component-Diagram.svg)

## **Cosmos chain - Component diagram**

## **Gravity contract - Component diagram**
