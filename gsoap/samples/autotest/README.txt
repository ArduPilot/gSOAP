
Autotest Server Feature Demonstration
=====================================

The soapcpp2 option -T generates an autotest server soapTester.c[pp], which can
be used to test a client application against, or to test messaging and XML
databindings. Option -T can be used in combination with option -i to generate
test server object classes, instead of C code.

In this example an autotest server is generated and used to verify the schema
mapping capability against 294 schema patterns published by the W3C Schema
Patterns for Databinding Working Group.

XML Schema Patterns for Databinding Interoperability Testing
------------------------------------------------------------

Both basic and advanced XML schema patterns test suite are passed by gSOAP.
For more information about the W3C working group databinding test patterns, see

    http://www.w3.org/2002/ws/databinding/

Code is autogenerated from WSDL and XSD documents for interoperability testing.
This example shows the auto test functionality for a large WSDL with 294 echo
operations for basic and advanced XML schema patterns. The generated auto test
server implements the echo operations where the echo operations are generated
by soapcpp2 option -T based on the fact that the input and output operation
types are identical.

From the 294 test patterns, gSOAP passes all except for 10 that are deemed 
impossible to test because these 10 patterns in examples.xsd/examples.wsdl
(provided by the working group) have errors due to skipped test patterns in the
schemas.

For details on errors and skipped tests, see further below.

Acknowledgments and Licenses
----------------------------

W3C XML Schema Patterns for Databinding Working Group:

    http://www.w3.org/2002/ws/databinding/

Schemas and XML documents duplicated here with permission:

    examples.wsdl			WSDL from the W3C (with some edits)
    examples.xsd			the schema only, provided by the W3C
    ChameleonIncluded.xsd		referenced in examples.wsdl
    Included.xsd			referenced in examples.wsdl
    RelativeIncluded.xsd		referenced in examples.wsdl
    redefineschema.xsd			referenced in examples.wsdl
    strict.xsd				referenced in examples.wsdl
    databinding/examples/6/09/.../	(copy of XML message files only)

W3C copyright and document licensing:

    http://www.w3.org/Consortium/Legal/copyright-documents

Introduction and Usage
----------------------

This example demonstrates the auto-test code generation features of gSOAP. An
echo server is auto-generated and tested with auto-generated XML request
messages and the W3C working group's databinding patterns.

1. Invoke the 'wsdl2h' tool to generate the C++ databindings:

    > wsdl2h -d examples.wsdl

    -d ensures we capture mixed content, anyType, anyAttribute.

   Other suggested options for databinding interoperability:

    -P to suppress anyType inheritance (to simplify code).
    -f to flatten type inheritance hierarchy, includes base members in classes.
    -c to generate ANSI C (but limits type inheritance).
    -s to remove STL dependence.
    typemap.dat addition to increase xsd:dateTime range (from time_t's limits):
      xsd__dateTime = #import "custom/struct_tm.h" | xsd__dateTime
    typemap.dat addition to map xsd:duration to a 64 bit long:
      xsd__duration = #import "custom/duration.h" | xsd__duration

2. Invoke the 'soapcpp2' tool to generate the C++ server logic and autotest:

    > soapcpp2 -SL -T -I../../import examples.wsdl

    -T to generate a test echo server.
    -S to generate server-side code only.
    -L to omit *Lib.cpp generation.

   This generates:

    soapStub.h		copy of examples.h without annotations
    soapH.h		serializers
    soapC.cpp		serializers
    soapServer.cpp	server operation dispatcher (RPC skeleton)
    soapTester.cpp	echo server operation implementations (-T option)

   Other suggested options for databinding interoperability:

    -s to enforce strict validation.

3. Compile:

    > c++ -o autotest soapTester.cpp soapServer.cpp soapC.cpp -libgsoap++.a

   or compile with stdsoap2.cpp and dom.cpp:

    > c++ -o autotest soapTester.cpp soapServer.cpp soapC.cpp stdsoap2.cpp dom.cpp

4. Run the tests:

    > ./autotest < SoapBinding.echoXYZ.req.xml

   Note: XYZ is one of the patterns used in the databinding test suite.

   The autotest server can also run as a standalone application on port 8080:

    > ./autotest 0 8080

   It will accept the test message over HTTP, e.g. using a router app.

   A shell script is included to test all gSOAP auto-generated XML request
   messages:

    > sh test-self.sh

   which generates a test-self.log with request-response messages.

   A shell script is included to test all W3C basic and advanced XML schema
   patterns with SOAP 1.1 and SOAP 1.2, respectively:

    > sh test-patterns11.sh
    > sh test-patterns12.sh

   which generate test-patterns11.log and test-patterns12.log with
   request-response messages. The faults are not filtered, so SKIPPED tests
   show up as faults in these log files (more details below).
	
Corrected errors and skipped tests in databinding test suite
------------------------------------------------------------

Note: we changed <xs:import schemaLocation="..."> in examples.wsdl to make all
imported XSD files local (rather than http addresses to download).

Some test patterns are "SKIPPED". These patterns are skipped due to the
incomplete examples.xsd schema:

1.  echoBlockDefault: SKIPPED

    <xs:element name="echoBlockDefault">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:blockDefault"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

2.  echoElementReferenceUnqualified: fixed in examples.wsdl:
    xmlns="http://www.w3.org/2002/ws/databinding/examples/6/09/" added

3.  echoFinalDefault: SKIPPED

    <xs:element name="echoFinalDefault">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:finalDefault"/> -->
    </xs:sequence>
    </xs:complexType>

4.  echoMinOccurs1: fixed in examples.wsdl: two value elements but maxOccurs=1,
    changed to 2

5.  echoMixedComplexContent: requires DOM support to pass (wsdl2h option -d)
    and by treating all content with a DOM (thus, removing other class members).

6.  echoMixedContentType: requires DOM support to pass (wsdl2h option -d)
    and by treating all content with a DOM (thus, removing other class members).

7.  echoNoTargetNamespace: SKIPPED

    <xs:element name="echoNoTargetNamespace">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:noTargetNamespace"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

8.  echoQualifiedLocalAttributes: SKIPPED
    incomplete:

    <xs:element name="echoQualifiedLocalAttributes">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:qualifiedLocalAttributes"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

9.  echoQualifiedLocalElements: SKIPPED

    <xs:element name="echoQualifiedLocalElements">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:qualifiedLocalElements"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

10. echoSOAPEncodedArray: SKIPPED

    <xs:element name="echoSOAPEncodedArray">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:soapEncodedArray"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

11. echoSchemaVersion: SKIPPED, because examples.xsd appears incomplete:

    <xs:element name="echoSchemaVersion">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:schemaVersion"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

12. echoTargetNamespace: SKIPPED

    <xs:element name="echoTargetNamespace">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:targetNamespace"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

13. echoUnqualifiedLocalAttributes: SKIPPED

    <xs:element name="echoUnqualifiedLocalAttributes">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:unqualifiedLocalAttributes"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

14. echoUnqualifiedLocalElements: SKIPPED

    <xs:element name="echoUnqualifiedLocalElements">
    <xs:complexType>
    <xs:sequence>
    <!-- <xs:element ref="ex:unqualifiedLocalElements"/> -->
    </xs:sequence>
    </xs:complexType>
    </xs:element>

Fixed test patterns that did not conform to the pattern schemas:

1.  echoComplexTypeAttributeExtension: fixed placement of gender attribute in
    test pattern databinding/examples/6/09/ComplexTypeAttributeExtension:

    <ex:complexTypeAttributeExtension
        xmlns:ex="http://www.w3.org/2002/ws/databinding/examples/6/09/"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:wsdl11="http://schemas.xmlsoap.org/wsdl/"
        xmlns:soap11enc="http://schemas.xmlsoap.org/soap/encoding/">
        <ex:name gender="female">Mary</ex:name>
    </ex:complexTypeAttributeExtension>

    Correct:

    <ex:complexTypeAttributeExtension
        xmlns:ex="http://www.w3.org/2002/ws/databinding/examples/6/09/"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:wsdl11="http://schemas.xmlsoap.org/wsdl/"
        xmlns:soap11enc="http://schemas.xmlsoap.org/soap/encoding/"
	gender="female">
        <ex:name>Mary</ex:name>
    </ex:complexTypeAttributeExtension>

2.  echoLocalElementSimpleType: fixed <ex:localElement>???</ex:localElement> in
    test pattern examples/6/09/echoLocalElementSimpleType:

