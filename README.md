# Bnd Chinese
1. Release
2. Introduction(Terry)
 * Introduction (Terry)
 * A Bit of History(Terry)
 * Transient Dependencies(Joye)
 * Packages(Joye)
 * BTool(Joye)
 * Eclipse(Joye)
 * Inside the OSGi Alliance(Joye)
 * Bndtools(Joye)
 * Manual(Joye)
3. How to install bnd
 * How to install bnd(Terry)
 * Command Line(Terry)
 * Libraries(Terry)
 * Source Code(Terry)
 * Manual(Terry)
 * Communication Settings(Terry)
4. Guided Tour
 * Guided Tour(Terry)
 * Workspace(Terry)
 * Naming(Terry)
 * Properties(Terry)
 * Plugins(Terry)
 * Project(Terry)
 * Setup(Terry)
 * Changing the Layout(Terry)
5. Guided Tour Workspace & Projects
 * Guided Tour Workspace & Projects(Joye)
 * Wrapping(Joye)
 * Simplistic(Joye)
 * Properties Format(Joye)
 * Running bnd(Joye)
 * Output(Joye)
 * The Manifest(Joye)
 * Export Package(Joye)
 * Private Package(Joye)
 * Bundle Version(Joye)
 * DNRY or the Benefit of Macros(Joye)
 * Description(Joye)
 * Include Resources(Joye)
 * Import Package(Joye)
 * Imports(Joye)
 * Remove Headers(Joye)
 * Includes(Joye)
 * Sub bnd Files(Joye)
 * Merging(Joye)
 * Alternative Wrap(Joye)
 * Errors & Warnings(Joye)
 * Failing(Joye)
 * Settings(Joye)
 * System Commands(Joye)
 * Upto(Joye)
 * Workflow(Joye)
6. Concepts
 * Concepts(Joye)
 * Bundles(Joye)
 * Components(Joye)
 * The Workspace(Joye)
7. Best practices
 * Workspace(Terry)
8. Build
 * Workspace Properties(Terry)
 * Extension Files(Terry)
 * Local customizations(Terry)
 * Workspace Plugins(Terry)
 * Project Properties(Terry)
 * Run instructions(Terry)
 * Launching(Terry)
 * Testing(Terry)
 * Overriding the plugins(Terry)
9. Generating JARs
 * Generating JARs
 * Project
 * Manifest
 * Extra entries on the Classpath
10. Versioning
 * Versioning
 * Best Practices
 * Versions in OSGi
 * Versioning Packages
 * When Package Version Differ
 * Import Version Policy
 * Substitution
 * Versioning Bundles
 * Why does bnd use only major and minor version component in import-package headers?
11. Baselining
 * Baselining
 * Setting Up a Project for Baselining
 * Example baselining Project Instructions
12. Service Components
 * Service Components
 * Components and Metatype
13. Metatype
 * Metatype
 * Naming
 * @Meta.OCD
 * @Meta.AD
 * Runtime conversions
 * Example
14. Contracts
 * Contracts
15. Manifest Annotations
 * Manifest Annotations
 * Manifest Annotations
 * Require & Provide Capability Manifest Annotations
 * RequireCapability
 * ProvideCapability
 * Example
 * Bundle License
 * Example
 * More Manifest Annotations
16. Resolving Dependencies
 * Resolving Dependencies
17. Launching
 * Launching
 * Launcher Architecture
 * biz.aQute.launcher
 * Example bndrun file
 * Packaging
 * Exit codes
 * Remote Launching
 * Parts
 * Usage
 * Example bndrun
18. Testing
 * Testing
 * Model
 * Framework Properties for the Default Tester
 * How to set the Test-Cases Macro Automatically
 * Continuous Testing
 * Other Tester Frameworks
 * Older Versions
19. Packaging Applications
20. Wrapping Libraries to OSGi Bundles
 * Wrapping Libraries to OSGi Bundles
 * Introduction
 * Initial Wrapping
 * Examining Dependencies
 * Refining Dependencies
 * Indicating Optional Dependencies
 * Package Level Used-By Analysis
 * Splitting the Bundle
 * Class Level Used-By Analysis
 * Excluding Imports
 * Versioning Imports
 * Other Concerns
 * Class References in Configuration
 * Summary
 * A Template
21. From the command line
 * From the command line
 * General Options
 * print ( -verify | -manifest | -list | - all ) * .jar +
 * buildx ( -classpath LIST | -eclipse | -noeclipse | -output ) * .bnd +
 * wrap ( -classpath ((',')*)-output <file|dir> | -properties ) * \\
 * Eclipse
 * Eclipse Plugin
22. For Developers
 * API
 * Creating a Manifest
23. Plugins
 * Plugins
24. Tools bound to bnd
 * Reference
25. File Format
 * Instructions
 * Types of Instructions
 * Parameters
 * Basic Types
 * Selectors
26. Header Reference
 * Reference
27. Instruction
 * Instruction
 * Syntax
 * Merged Instructions
 * SELECTOR
 * FILE
 * FILESPEC
 * PATH
 * GLOB
28. Instruction Index
 * Instruction Index
29. Macro Reference
 * Macro Reference
 * Macro patterns
 * Arguments
 * Wildcarded Keys
 * Expansion of ./
 * Booleans
 * Types
 * Reference
30. Command Reference
 * Command Reference
 * Reference
31. Plugins Reference
 * Reference
32. Settings
 * Settings
 * Authorization
 * The bnd settings Command
 * The global macro
 * Authorizing a New System
33. Errors
34. Warnings
 * Java source version inconsistency: bnd has ‘1.5’ while Eclipse has ‘1.8’. Set the bnd ‘javac.source’ property or change the Eclipse project setup.
35. Frequently Asked Questions (Joye)
 * Frequently Asked Questions
 * How to ask a question 
 * Too Many Imports 
 * Importing the default package error 
 * Remove unwanted imports 
 * No Imports Show Up 
 * Why No Automatic Bundle-Activator 
 * How to assign an unbind method to a @Reference? 
 * packageinfo or package-info.java? 
 * Why are superclass not inspected for Component annotations? 
 * Can’t find the source of the version on an Export-Package? 
 * Should I use the Bundle-ClassPath? 
 * What should I use instead of the Bundle-ClassPath? 
 * Sharing CNF Folder and BNDTools Projects 
