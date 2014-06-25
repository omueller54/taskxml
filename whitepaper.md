#An XML exchange format for (programming) tasks

**Version 0.9.2**

contributors listed in alphabetical order:


Humboldt-Universität zu Berlin: Niels Pinkwart, Sven Strickroth  
Technische Universität Clausthal: Oliver Müller  
Universität Duisburg-Essen: Michael Striewe  
Hochschule Hannover: Sebastian Becker, Oliver J. Bott  
Universität Osnabrück: Helmar Gust, Nadine Werner  
Ostfalia Hochschule für Angewandte Wissenschaften: Stefan Bisitz, Stefan  
Dröschler, Nils Jensen, Uta Priss, Oliver Rod  

[TOC]

##Introduction

This document specifies syntax and semantics for a standardized “Task
Format” - an exchange format for programming exercises/tasks. It
formalises the specification of programming exercises, for example, the
description, required programming language and suggested tests for
evaluating the student submitted code so that exercises written for one
tool can be exported and imported into another tool. The main
e-assessment tools considered for this format are Praktomat, VIPS, JACK,
GATE or Moodle-Grapper WS (inspired by Web-CAT) most of which are
included in the eCULT project “ProFormA”. The latest version of this
document and the corresponding XML Schema can be found at this address:
http://go.ecult.me/13022046A

##General Structure

The XML exchange format consists of two parts: The first section shows
the description/specification of a task, including supporting files; and
the second section demonstrates a specification of tests which are to be
included in the specification of a task under the \<tests\> tag. Each
task can have many tests.

##Format specification

The XML file can be exchanged as a standalone file. However, if it
refers to several files, it is recommended to combine the XML file and
all linked files into a ZIP-archive. The XML file itself should be
stored in the root of the ZIP under the name “task.xml” so that it can
be found by tools supporting this format. The XML file should be well
formed and encoded in UTF-8 (RFC 3629). ID’s are globally unique within
the XML document.

##Task section

###XML Specification

The general structure of the XML format is given as follows (this is
meant to provide an overview and does not represent a minimal document):

    <task lang="[LANG code]" xmlns="urn:proforma:task:v0.9.2">
    <description></description>
    <proglang version=""></proglang>
    <submission />

    <files />

    <model-solutions />

    <tests />

    <grading-hints />

    <meta-data />
    </task>


The following code shows the XML Schema for the Task Format:

    <xs:element name="task">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="description"/>
                <xs:element ref="proglang"/>
                <xs:element ref="submission"/>
                <xs:element ref="files"/>
                <xs:element ref="model-solutions"/>
                <xs:element ref="tests"/>
                <xs:element ref="grading-hints" minOccurs="0"/>
                <xs:element ref="meta-data"/>
            </xs:sequence>
            <xs:attribute name="lang" type="xs:string" use="required"/>
        </xs:complexType>
            <xs:keyref name="tests-filerefs-fileref" refer="fileid">
             <xs:selector xpath="tests/test/test-configuration/filerefs"/>
             <xs:field xpath="fileref"/>
         </xs:keyref>
    </xs:element>

The document root element “task” holds the XML-namespace URI for the
current version number of the XML Task Format. The only currently valid
value is “urn:proforma:task:v0.9.2”. The task itself must have an
attribute “lang” which specifies the natural language used. The
description, title etc should be written in this language. The content
of the “lang” attribute must comply with the IETF BCP 47, RFC 4647 and
ISO 639-1:2002 standards.

###The description part

    <xs:element name="description"  type="xs:string" />

An instance of this element contains the task description as text. A
subset of HTML is allowed (see Appendix A).

###The proglang part

   <xs:element name="proglang">
     <xs:complexType>
         <xs:simpleContent>
             <xs:extension base="xs:string">
                 <xs:attribute name="version" type="xs:string" use="required"/>
             </xs:extension>
         </xs:simpleContent>
     </xs:complexType>
   </xs:element>

An instance of this element contains the programming/modelling/query
language to which this task applies. A valid list of values is specified
in Appendix B. The “version” attribute specifies which version of the
language was used in the creation of the task. (The task is guaranteed
to work with that version – any other requirements about version
compatibility must be checked externally.)  The “version” must be
entered as a “point” separated list of up to four unsigned integers.

###The submission part

    <xs:element name="submission">
        <xs:complexType>
            <xs:attribute name="max-size" type="xs:positiveInteger" use="optional"/>
            <xs:attribute name="allowed-upload-filename-regexp" type="xs:string" use="optional" default=".*"/>
            <xs:attribute name="unpack-files-from-archive" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="unpack-files-from-archive-regexp" type="xs:string" use="optional" default=".*"/>
        </xs:complexType>
    </xs:element>

An instance of this element can specify restrictions for the upload of
(solution) files - by default there are no restrictions. The possible
restrictions can be entered into a set of four optional attributes:
“max-size”, “allowed-upload-filename-regexp”,
“unpack-files-from-archive”, and “unpack-files-from-archive-regexp”. A
prefix of “\^” and a postfix of “\$” is implicitly assumed and must not
be specified explicitly.

-   “max-size” specifies the maximum size of a file in bytes which
    should be accepted. Systems which have a stronger limit of the file
    size should print a warning to the importing user. If this attribute
    is missing, a system default value will be used. 
-   “allowed-upload-filename-regexp” holds a regular expression of the
    filenames (only the filename, without path) which the system should
    accept. 
-   "unpack-files-from-archive" specifies if uploaded archives (zip/jar)
    should be unpacked automatically. If it is set to *false,* no
    extraction takes place and the archive is used as it is. 
-   In case "unpack-files-from-archive" is set to *true,*
    “unpack-files-from-archive-regexp” is honoured which holds a regular
    expression that controls which files are automatically extracted
    (the filename of the uploaded archive must of course match
    “allowed-upload-filename-regexp”). Only matching files (the whole
    path of the zip-items matches with “/” as path separator) are
    extracted from the archive.

###The files part

    <xs:element name="files">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="file"/>
            </xs:sequence>
        </xs:complexType>
            <xs:key name="fileid">
             <xs:selector xpath="file"/>
             <xs:field xpath="@id"/>
         </xs:key>
    </xs:element>

The files element contains 0 or more file elements. A file element is
used to attach files to a task. Files can be external or embedded into
the XML file.

###The file element

   <xs:element name="file">
     <xs:complexType>
         <xs:simpleContent>
             <xs:extension base="xs:string">
                 <xs:attribute name="id" type="xs:string" use="required"/>
                 <xs:attribute name="filename" type="xs:string" use=”required"/>
                                                <xs:attribute name="comment" type="xs:string" use=”optional"/>
                 <xs:attribute name="class" use="required">
                     <xs:simpleType>
                         <xs:restriction base="xs:string">
                             <xs:enumeration value="template"/>
                             <xs:enumeration value="library"/>
                             <xs:enumeration value="workingdata"/>
                             <xs:enumeration value="instruction"/>
                             <xs:enumeration value="internal"/>
                         </xs:restriction>
                     </xs:simpleType>
                 </xs:attribute>
                <xs:attribute ref="type" default="embedded"/>  
             </xs:extension>
         </xs:simpleContent>
     </xs:complexType>
   </xs:element>
   <xs:attribute name="type">
     <xs:simpleType>
         <xs:restriction base="xs:string">
             <xs:enumeration value="file"/>
             <xs:enumeration value="embedded"/>
         </xs:restriction>
     </xs:simpleType>
    </xs:attribute>

The file element includes or links a single file to a task. Each
instance/file must have a (task) unique string in its “id” attribute (in
order to reference this file within this task) and has to be classified
using the “class” attribute with one of the following values:

-   “template”: The file is a template for students to be used as a
    starting point for their solution.
-   “library”: The file is a library to be used by students (and might
    also be required for tests).
-   “inputdata”: The file contains data which the algorithm of the
    students should work with.
-   “instruction”: The file contains further instructions for handling
    the task, e.g. an UML activity diagram.
-   “internal”: This file is not visible for students and holds files
    which are required for processing the task/tests within the system.


The information about how a file is used in a test is supplied by the
test-configuration which uses a file (via its ID). Further details can
be provided in the optional “comment” attribute. The file itself can be
embedded into the XML (recommended for shorter plain text files) or can
be included in the ZIP-archive (recommended for binary files). If a file
is embedded, the “type” attribute must be set to “embedded” and the text
content of the element is the file content. The “filename“ attribute has
to be set to define a filename (e.g., for Java classes where the class
name must be equal to the filename, it can also include a relative
path). This attribute defines the name the file will have when it is
executed. If the name is unimportant, a default name can be used. If a
file is not embedded, the “type” attribute must be set to “file” and the
text content of the element contain the filename within the task ZIP
archive (which can be different from the filename attribute).



###The model-solutions part

    <xs:element name="model-solutions">
        <xs:complexType>
            <xs:sequence maxOccurs="unbounded">
                <xs:element ref="model-solution"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

The model solutions element is used to provide one or more solutions of
the task. For each model-solution a new model-solution element is added.

###The model-solution element

    <xs:element name="model-solution">
        <xs:complexType>
            <xs:simpleContent>
                <xs:extension base="xs:string">
                    <xs:attribute name="id" type="xs:string" use="required"/>
                    <xs:attribute name="filename" type="xs:string" use="required"/>
                                                   <xs:attribute name="comment" type="xs:string" use=”optional"/>
                    <xs:attribute ref="type" default="embedded"/>
                </xs:extension>
            </xs:simpleContent>
        </xs:complexType>
            <xs:unique name="model-solutionid">
             <xs:selector xpath="model-solution"/>
             <xs:field xpath="@id"/>
         </xs:unique>
    </xs:element>

The model-solution element includes or links a single model-solution to
a task. Each instance/solution must have a (task) unique string in its
“id” attribute. The model-solution can be directly embedded into the XML
file or attached in an extra file within the zip archive (cf. “The file
part”). The optional attribute “comment” can be used for additional
information, for example if more than one model solution is provided it
can be explained why there are several solutions.

###The tests part

The tests element is used to provide automatic checks and tests for the
task. More specific information about the test XML is provided in the
[second section](#id.gmls3d60jimk) of this paper.

##The grading-hints element##

    <xs:element name="grading-hints">
        <xs:complexType>
         <xs:sequence>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>

An instance of this element holds information on how the creator of the
task intended the grading process. This element contains plain text or
arbitrary elements in different namespaces. This field is mainly
intended to support an exchange of grading ideas and also to allow tasks
to be exported and imported again from one system to another.

###The meta-data element

    <xs:element name="meta-data">
        <xs:complexType>
         <xs:sequence>
             <xs:element ref="title"/>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>

The meta-data element holds a namespace for the meta-data of each
system. Because meta-data are already standardized in other systems, it
was decided not to formalize them as part of this specification. Only
meta-data relevant for the whole task should be entered here. Meta-data
that is specific to an individual test should be entered in the
test-meta-data element.

##Test section

###XML Specification

The general structure of the test description is given as follows:

    <tests>
        <test id="unique ID" validity="">
            <title></title>
            <test-type></test-type>
            <test-configuration>
                <filerefs>
                    <fileref></fileref>
    </filerefs>
            </test-configuration>
                  <test-meta-data />
    </test>
    </tests>

The corresponding XML schema for the test XML structure is:

    <xs:element name="tests">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                <xs:element ref="test"/>
            </xs:sequence>
        </xs:complexType>
            <xs:unique name="testids">
             <xs:selector xpath="test"/>
             <xs:field xpath="@id"/>
         </xs:unique>
    </xs:element>

###The test element

    <xs:element name="test">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="title"/>
                <xs:element ref="test-type"/>
                <xs:element ref="test-configuration"/>
            </xs:sequence>
            <xs:attribute name="validity" use="optional" default="1.00">
                <xs:simpleType>
                    <xs:restriction base="xs:decimal">
                        <xs:totalDigits value="3"/>
                        <xs:fractionDigits value="2"/>
                        <xs:minInclusive value="0"/>
                        <xs:maxInclusive value="1.00"/>
                    </xs:restriction>
                </xs:simpleType>
            </xs:attribute>
            <xs:attribute name="id" use="required" type="xs:string"/>
        </xs:complexType>
    </xs:element>

The test element has a required attribute “id” and an optional attribute
“validity”. The optional attribute “validity” is used by some systems
(such as Vips) for tests which only partially verify the solution code.

###The title element

    <xs:element name="title" type="xs:string"/>

The title element is used to provide a short and clear name for the test
that can be displayed to students as part of their results. It should be
noted that the title does not have a language attribute because it is
assumed that the title is written in the same natural language as
specified for the task itself.

###The test-type element

    <xs:element name="test-type" type="xs:string"/>

Examples of values are: java-syntax, java-junittest3. A list of allowed
entries is specified in Appendix C.

###The test-configuration part

    <xs:element name="test-configuration">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="filerefs" minOccurs="0"/>
                <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
                <xs:element ref="test-meta-data" minOccurs="0"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

The test-configuration contains all parameters which are needed for
configuring this specific test. The test-configuration should either
contain a files part or a code element which contains the actual code of
the test. It has sub-elements which are required, however, each test can
also have elements of its own namespace for test-type specific
configuration options.

###The filerefs part

    <xs:element name="filerefs">
        <xs:complexType>
            <xs:sequence minOccurs="0" maxOccurs="unbounded">
                    <xs:element ref="fileref"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

Several filerefs can be specified via file elements.

###The fileref element

    <xs:element name="fileref" type="xs:string"/>

The fileref element links a single file to a test based on the ID of the
file which has to be defined in task/files. The ID has to be entered as
the simple content of the fileref element.

###The test-meta-data element

    <xs:element name="test-meta-data">
        <xs:complexType>
         <xs:sequence>
             <xs:any namespace="##other" minOccurs="0" maxOccurs="unbounded"/>
         </xs:sequence>
        </xs:complexType>
    </xs:element>

The test-meta-data element holds a namespace for test-specific meta-data
of each system. This is particularly useful for attributes that are
required for ex- and import in one system but which are not relevant for
other systems.

##Appendix A: Subset of HTML

<\!\-\- … \-\-\\>, a, b, blockquote, br, p, sup, sub, center, div, dl, dd,
dt, em, font, h1, h2, h3, h4, h5, h6, hr, img, li, ol, strong, pre,
span, table, tbody, td, tr, th, tt and ul.

However, for div, b, br, dd, dt, dl, blockquote, p, sup, sub, h1, ...h6,
li, ol, hr, strong, pre, span, table, tbody, td, tr, th, tt and ul there
are no attributes allowed in order to avoid problems with different
layouts.

##Appendix B: List of programming languages

-   java
-   SQL
-   prolog

##Appendix C: List of test types

-   java-compilation
-   java-junit3
-   java-checkstyle
-   java-code-coverage-emma
-   java-findbugs
-   java-pmd
-   dejagnu
-   anonymity (heuristics for checking that students have not included
    their names in the code)
