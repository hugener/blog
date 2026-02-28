---
title: Revisiting the test automation pyramid
date: 2024-03-09
tags: UI Automation, maintainable tests, BDD, test pyramid
---
## Introduction

### Software testing

Automated testing has become essential to modern software development, enabling rapid feedback cycles and continuous delivery while maintaining quality. The test automation pyramid, introduced by Mike Cohn, addresses a critical challenge: how to distribute testing efforts across different levels of abstraction to maximize coverage while minimizing maintenance burden and execution time. The pyramid advocates for a broad base of fast, isolated unit tests, a middle layer of integration tests, and a narrow top of UI tests.

The test automation pyramid has served as a guideline for development teams for decades, recommending a distribution of test types that favors lower-level tests with minimal integration over UI/E2E tests.

<figure style="text-align:center">

![the test automation pyramid](Revisiting%20the%20test%20automation%20pyramid/Revisiting%20the%20test%20automation.png)

</figure>
<p align="center">
    <a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a>
</p>

<a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a> identifies the following limitations of UI tests:

<figure style="text-align:center">

![problems with ui tests](Revisiting%20the%20test%20automation%20pyramid/Problems%20with%20UI%20tests.png)

</figure>

<p align="center">
    <a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a>
</p>

While the difficulties described by the test automation pyramid are valid, this article challenges the practice of minimizing UI tests and arguing that untested integration lead to bugs. In particular mission critical systems, this practice causes a heightened burden of manual testing. Solving this problem has to provide solutions to the three above mentioned problems, brittleness, expensive to write and time consuming.

### Stable tests

In his book, Cohn goes on to state that not all test cases need to be run through the user interface <a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a> and that this is where the service layer comes in <a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a>. In other words, the tests written against this level are not susceptible to the same level of brittleness as on the UI level.

For the purposes of this article, tests at this level are considered equivalent to BDD-style tests, acceptance tests, or system tests, despite some inconsistencies in their original definitions. Due to the stability of this layer, it is an excellent candidate for writing tests against and helps creating a consistent vocabulary for tester, analyst, developers and the business <a href="#Introducing%20BDD">(North D.,2006)</a>.

Service-level tests are also very useful to create a mapping between map specifications, allowing quality management.

### Utilizing the compiler

Compiler checks and guarantees is a heavily researched topic, which constantly pushes more work to be done at compile time. Over the years, languages and compilers have evolved to include more compile time guarantees about the actual implementation of a program. Many of these are found in functional programming languages approach to using type systems to provide such guarantees. Another example is Rust memory management, where the Rust compiler enforces rigid rules of ownership, and thus provides a guarantee of correct memory management.

This article explores whether the problems with UI testing can be solved by a similar approach using compiler features to ensure the automatability of an application.

## How to run service-level tests against the UI?

In traditional BDD, there is a specification, which is connected to some test or automation code, which again execute against the application. Whether the specification is written in business readable language or something more formal like Gherkin is not important.

In the diagram below, testing the UI and business logic is separated by a horizontal line and generally the business logic part is equivalent to service-level tests.

The automation code acts as an intermediary layer between test specifications and the application.

*   **UI automation bindings** - Code that translates high-level test steps into specific UI interactions (clicks, text entry, element location)
*   **Business logic bindings** - Mappings between domain concepts in tests and the application's internal business logic

Applying the same approach at the UI level introduces the previously discussed challenges.

<figure style="text-align:center">

![traditional bdd](Revisiting%20the%20test%20automation%20pyramid/Traditional%20BDD.png)

</figure>

The proposed approach separates automation code between the application and an automation runner framework, leveraging compiler capabilities. The application architecture—particularly the service layer (henceforth referred to as the Use Case layer) and UI layer—is designed to enable automatic extraction of mappings between these layers. In the diagram below, the application component is depicted as larger to reflect the additional metadata it exposes—specifically, mapping information between the Use Case and UI layers. Information that previously resided in external automation code is now embedded directly into each application build.

The remaining automation code can be generalized into a reusable framework that, once developed, is stable, reusable, and application-agnostic.

This framework supports two execution modes:

1.  **Use Case mode**: The test runner invokes the Use Case layer directly, equivalent to traditional BDD-style tests.
2.  **UI mode**, where the application's Use Case mapping is used to discover Automation-IDs to stimulate the application via an automation client.

<figure style="text-align:center">

![google maps for UIs](Revisiting%20the%20test%20automation%20pyramid/Google%20maps%20for%20UIs.png)

</figure>

**Automation-IDs** are identifiers automatically generated and assigned to UI elements and derived from the either from Use Case operations or ViewModel operations those elements they trigger.

**Use Case mapping (Automation Metadata)** refers to information embedded directly within the application that describes the relationships between Automation-IDs and Use Case operations. By encoding this mapping at the architectural level, automation frameworks can interact with the application without relying on hardcoded element locators — eliminating a primary source of test brittleness.

## The application layer

To implement this concept, the application layer is divided into two distinct layers. The UI layer and the Use Case layer.

The UI layer contains all state, interactions, and views needed to build a functional UI.

The Use Case layer rather represents the application's API and Use Cases do not map one-to-one to UI interactions. It instead represents what is possible in the application from a programmatic perspective, remaining agnostic to how these capabilities are surfaced in the UI. A clean abstraction from the UI layer is essential for producing stable tests.

Another important aspect is that the Use Case layer, cannot depend on the UI layer at runtime. This means it cannot depend on an event to be triggered by the UI other than those caused by the user, because this will break running tests in Use Case mode. 

<figure style="text-align:center">

![application layer](Revisiting%20the%20test%20automation%20pyramid/Application%20layer.png)

</figure>

The conceptual diagram below shows that the application can be implemented through the Model-View-ViewModel (MVVM) pattern, where the models offer various steps. These steps are mapped in the ViewModel, where they are prepared for presentation in the view.

Use Cases are not modelled as explicit system components but instead emerge through user interactions with the View.

<figure style="text-align:center">

![conceptual model](Revisiting%20the%20test%20automation%20pyramid/Conceptual%20model%20diagram.png)

</figure>

## Implications

*   No one-to-one mapping exists between the model API and the steps exposed by the ViewModel. Consequently, multiple steps through the View and ViewModels may correspond to a single API call on the model.
*   Automatic assignment of Automation-IDs requires that both Steps and API endpoints be uniquely identifiable, with Automation-IDs derived from these identifiers.

### Limitations

The concept does introduce some limitations

*   **Complete code coverage cannot be achieved**: Since tests can only be expressed through the Use Case layer, UI paths where a single Use Case step is accessible from multiple UI entry points cannot be tested without specifying exact UI navigation. However, specifying explicit UI paths contradicts the goal of maintaining UI-agnostic tests.
    
    ➡️ From a user experience perspective, whether providing multiple pathways to identical functionality constitutes good design remains debatable.
    
    ➡️ An alternative approach would implement a path variation mode, where tests execute using non-optimal/alternative navigation routes in addition to shortest paths. This strategy enables higher coverage through multiple test executions without introducing brittleness.
    
    Even without implementing these strategies, achieving high UI layer coverage should remain feasible.
*   **Application to legacy systems remains unclear**: This approach is most readily applicable to greenfield projects. The methodology for incorporating new functionality into legacy systems while leveraging this automation framework requires further investigation. Any migration of existing systems would likely necessitate beginning with core functionality.

## The testing cup

With this framework established, <a href="#Succeeding%20with%20Agile%20Software">(Cohn, 2010)</a>'s test automation pyramid can be reconsidered.

*   **Time consuming** - Test runtimes remain substantial; the primary mitigation strategy involves test-suite partitioning and parallel execution across multiple test agents. While not specifically addressed by this framework, scaling test infrastructure is readily achievable with contemporary cloud platforms.
*   **Brittleness and expensive to write** - This framework addresses both limitations simultaneously. From an implementation perspective, Service-level and UI-level tests are structurally identical. In other words, Service-level tests function as UI tests, thereby reducing both brittleness and development time.

The test pyramid thus represents a specific subset of tests executed as a quality gate for builds merging to the main branch. This subset enables merges within reasonable timeframes; the number of included tests scales with the degree of partitioning and parallelization implemented.

Consequently, the complete test suite more closely resembles a cup shape, superficially similar to the testing trophy proposed by <a href="#Static%20vs%20Unit%20vs%20Integration%20vs%20E2E">(Dodds, 2021)</a>. However, whereas the trophy's shape derives from prioritizing integration tests for return on investment, the shape proposed here emerges from a fundamentally different mechanism: the architectural embedding of automation metadata that allows service-level tests to execute through the UI.

<figure style="text-align:center">

![testing cup](Revisiting%20the%20test%20automation%20pyramid/Testing%20cup.png)

</figure>

The narrower base of the diagram reflects the principle of avoiding unit tests of implementation details, thereby mitigating the Fragile Test Problem <a href="#Test%20Contra-variance">(Martin, 2017)</a>, rather than being a direct consequence of the framework presented here. This aspect, however, falls outside the scope of the current discussion.

## Conclusion

This paper has presented a framework that fundamentally reimagines the relationship between test automation and application architecture. By embedding automation metadata directly into the application through compiler-generated mappings between the Use Case and UI layers, this approach addresses the core limitations Cohn identified with UI testing.

Future research should investigate how artificial intelligence could enhance the quality assurance of systems built using this framework. Given that each application maintains comprehensive metadata about its capabilities, training AI systems to conduct exploratory testing appears feasible.

## References


<a id="Succeeding%20with%20Agile%20Software"></a>
Succeeding with Agile Software Development Using Scrum  
Cohn, M. (2010). Succeeding with Agile Software Development Using Scrum. Addison-Wesley Professional.

<a id="Introducing%20BDD"></a>
Introducing BDD  
North, D. (2006). Introducing BDD. https://dannorth.net/introducing-bdd


<a id="Static%20vs%20Unit%20vs%20Integration%20vs%20E2E"></a>
Static vs Unit vs Integration vs E2E Testing for Frontend Apps  
Dodds, K. C. (2021). Static vs Unit vs Integration vs E2E Testing for Frontend Apps. kentcdodds.com. https://kentcdodds.com/blog/static-vs-unit-vs-integration-vs-e2e-tests

<a id="Test%20Contra-variance"></a>
Test Contra-variance  
Martin, R. C. (2017). Clean Coder Blog https://blog.cleancoder.com/uncle-bob/2017/10/03/TestContravariance.html