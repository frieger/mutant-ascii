# MUTANT: Model-Driven Unit Testing using ASCII-art as Notational Text #

MUTANT aims to make test developers' lives easier. Its key idea to specify test models using a text-based visual notation. The specifications are provided by annotating test methods, equipping test developers with the full benefits of the powerful editing and collaboration tools found in state-of-the-art source code IDEs.

The main component of MUTANT is a compiler that builds models from the visual specifications and makes these models available to the test framework through a dedicated API. To integrate the compiler in your existing tool chain, please follow the instructions provided below.

## Example ##
```Java
public class RefactoringTest extends UnitTest {
 @Test
 /** @InputModel EPackage pkg = 
       
                 +------------+                               
                 |   Person   |    										
                 +------------+                               
                     A   A                          
            .--------'   '-------.                 
            |                    |                 
     +--------------+   +--------------+      
     | Professor    |   | Student      |
     |--------------|   |--------------|      
     | name: String |   | name: String |      
     +--------------+   +--------------+   
   */
  public void testRefactoring() {
    EPackage pkg = Mutant.getPackage("pkg");
    EClass person = pkg.getEClass("Person");	
      	
    assertTrue(person.getAttributes().size()==0);
    PullUpRefactoring refac = new PullUpRefactoring(pkg);
    refac.execute();
    assertTrue(person.getAttributes().size()==1);
   }    
 }
 ```

The module under test in this example is a model transformation implementing the [_pull up attribute_](http://refactorings4gts.wikia.com/wiki/Pull_Up_Attribute) refactoring. The test model contains three classes, two with an equivalent attribute that is to be pulled up to the common superclass. In the test code, the specified test model is loaded using the dedicated API. The transformation is executed. Assertions are evaluated to check whether the attribute was indeed pulled up.

## How to run the example ##

Place the `MutantExample` project into your workspace and MUTANT should generate the model files for all contained test classes. You can verify this by deleting the files in the `mutant/` directory and rebuilding the `MutantExample` project.

### Building ###

We already configured a builder, so MUTANT should run automatically when the project is built. If you are not running Eclipse on Windows or want to use MUTANT for other projects, you will have to adapt the build configuration:

####  Configuring an Eclipse builder ####
1. In order to configure an Eclipse builder, right click on a project and select "Properties."
2. In "Builders," click on "New..." and select "Program."
3. In the "Main" tab, put the compiler (`MutantCompiler`) in "Location." Many platforms (including Windows), however, will disallow anything but an executable here. To this end, we provide a batch script that will call `java -jar PATH_TO_MUTANTCOMPILER/mutantcompiler.jar`. If you are on Windows, copy `mutantcompiler.bat` and `mutantcompiler.jar` from the `MutantExample/compiler` directory to some arbitrary directory on your hard drive and enter the location of the copied `mutantcompiler.bat` file into "Location." If you are on other platforms, you might have to write a very short script that calls java -jar yourself.
3. In "Arguments", enter the location of your Java source files (and optionally, the location of your meta-models). If you are using Eclipse, a convenient shortcut is `${build_project}` for the current project. *Note:* The example project uses `${build_project} ${build_project}/model`.
4. In the "Refresh" tab, you might want to select "Refresh resources upon completion."
5. The MUTANT builder should now run when you build the project. It is not optimized, so it might take a short while and produce a lot of debug output, which can be ignored.
6. The generated models should then be in the `mutant/` subdirectory.

#### Manual building ####

For the full experience, we  recommend you to configure a dedicated builder as described above. Howver, to get a first impression of what the compiler does, you can also trigger `MutantCompiler` manually. It takes one to three arguments:

1. The first argument is the location of your Java source files or the location of the project.
2. The second argument is optional: It is the location of your meta-models.
3. The third argument is the flag "DEBUG", which enables more verbose output.

MUTANT will create the `mutant/` subdirectory in the directory given by the first argument and then put models into this directory.


## Repository structure ##
This repository comprises the following Eclipse projects:

1. *MutantAsciiCompiler* parses annotated Java files and builds models
2. *MutantAsciiApi* provides easy access to these models from your Java application
3. *MutantExample* is an example of how the system can be used. See later for information on how to get the example running on your system.

### MutantAsciiCompiler ###
The compiler should be put into your build script. See later on how to get this to work using Eclipse.
The compiler is invoked by calling `mutant.main.MutantCompiler.main(String[] args)`. Arguments are: `Java-Source-Location [Model-Location] [DEBUG]`

 - `Java-Source-Location` specifies a directory with all your java source files. This directory will be traversed recursively.
 - `Model-Location` specifies a directory with all your meta-models (for abstract syntax). This directory will not be traversed recursively.
 - `DEBUG` is a flag that enables more verbose debug output.

**Attention** MUTANT will create the directory `mutant/` as a subdirectory of Java-Source-Location and will overwrite its contents.

### MutantAsciiApi ###
This provides some methods for getting the MUTANT-created models in your test cases. See the example project for usage.

### MutantExample ###
This is an example project demonstrating MUTANT.

Concrete syntax for classes and abstract syntax are demonstrated. In order to facilitate execution, we have packaged the MUTANT compiler in the directory "compiler". This is not required for usage, but makes the build script easier.

 - *PullUpRefactoring.java*: This is a class that implements a "pull up attribute" refactoring.
 - *PullUpRefactoringTests.java*: Implements the test from the paper using MUTANT's support for concrete syntax for class diagrams. MUTANT will convert primitive Java types (and String and Object) to the corresponding ECore ETypes for added convenience.
 - *Printer.java*: This is a class that implements methods to print model elements. (WIP)
 - *PrinterTest.java*: Implements tests for Printer.java. Demonstrates abstract syntax. (WIP)

## Features ##

### currently implemented ###
- Concrete syntax for class models
- Abbreviated edges
- Names and multiplicities
- Abstract syntax: Mostly, see below

### not implemented ###
- Element multiplicity (e.g. [n=3])
- Support for arbitrary EDataTypes as values in abstract syntax (currently supported: EENumLiterals, EString, EInt, EDouble, EFloat, EChar, EBoolean)

## Required libraries ##
MUTANT requires EMF and Ecore.
For your convenience, we include the required EMF and Ecore libraries (which are licensed under the Eclipse Public License Version 1.0) in the `lib/` directory.

## Editing tools
 - Eclipse's [block selection mode](http://blog.jooq.org/2013/10/12/eclipses-awesome-block-selection-mode/) is a nice tool for keyboard-based ASCII-art editing. It supports to add or remove characters in successive lines.
 - [ASCII Flow](http://asciiflow.com/) is a free online editor  providing mouse-based editing tools such as box drawing, line drawing, and a freehand eraser.
