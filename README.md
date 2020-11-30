# XInvoiceTools
BBj Utility Classes around the xInvoice and PDF/A standard

It is intended to demonstrate how to 
* create a PDF in PDF/A Standard using the various BBj-provided ways to print
* embed the XML invoice data to comply with EU eInvoice and German ZUGFERD standards

# Creating PDF/A for BBj Output

The BBjPrintSamples contains three subdirectories, one for each discussed path. (Contribution to improving these samples currently highly welcome!)

Demo.bbj shows the following three types of BBj output to PDF, and conversion to PDF-A 1b:

## BBJasper

Sample: Demo/PDFA_Jasper

For BBJasper, the success is only on the JasperReports side, which can do PDF/A directly. The following two sources describe the steps we used in this sample:

* https://community.jaspersoft.com/wiki/pdf-support-jasperreports-library
* https://stackoverflow.com/questions/36306170/how-can-i-export-report-to-pdf-a-1a-pdf-a-1b#new-answer
* https://supportdownloads.adobe.com/detail.jsp?ftpID=4077

For this very simple example, it came down to adding the following three lines to the invoice.jrxml:

```xml
<property name="net.sf.jasperreports.export.pdfa.conformance" value="pdfa1b" />
<property name="net.sf.jasperreports.export.pdfa.icc.profile.path" value="xInvoice/icc/AdobeRGB1998.icc" />
<style name="default" isDefault="true" fontName="DejaVu Sans"/>
```

The absolute icc file path is determined by the program in these lines:

```bbj
rem "SET THE ICC FILE PATH INTO THE REPORT"
iccpath$= BBjAPI().getFileSystem().resolvePath("../../icc/AdobeRGB1998.icc")
report!.getJasperReport().setProperty("net.sf.jasperreports.export.pdfa.icc.profile.path",iccpath$)
```

## BBjForm and SYSPRINT 

These two produce PDF which are almost okay, but miss a few metadata / header fields. The PDFA class adds these, which is shown in this demo. 

## Note about Unform

We have successfully tested the PDFA - class on output from the Unform product. It appears to work nicely and adds all necessary meta data, also for older versions. If somebody has Unform output which does not fully validate, open an issue and attach the PDF file (or send it offline if it contains sensitive data).

## Troubleshooting Fonts 

In case you're seeing validation problems, where the validator is nagging about a font (often Helvetica) not being embedded correctly, it's likely that the font you actually wanted to see in the PDF could not be found at creation time.

For BBj make sure to set the PDF Font Directory to the directory where you gave your fonts installed. Make sure to put all the .otf font files needed for generation in this directory.


For Linux, you can use some online TTF-to-OTF Converter, in case you're using Windows-Fonts like Arial.ttf in your printouts.


See https://documentation.basis.com/BASISHelp/WebHelp/sysguicontmethods/bbjapi/bbjapi_getbbjpdfform.htm.


Also make sure to recognize https://www.basis.com/content/kb-removal-fonts-jdk-11 for the removal of Fonts in OpenJDK 11.


# Validating PDF/A

The "validation" subdirectory contains a validator "PDFBox_PDFA_Validation.bbj" that iterates all PDFs in the "output" directory and creates a print preview listing the results. This validator is using the PDFBox library that ships with BBj.


The program Demo/PDFAValidator.bbj dynamically loads the jar files from the lib directory, so you do not have to add them to the classpath to just run this demo. In our testing on a BBj 20.30 development build, some of them did conflict with the shipped PDFBox jars, so we do not recommend to add them all to your production classpath, but run the validator always with a separate classpath if needed. (This may change in the future.)

#ZUGFeRD / xRechnung

There are several ways how you could create and merge the necessary XML into the PDF/A. This plug-in shows only one for now:

## using mustangproject.org

The Demo/MustangWriter.bbj program is basically the transcript in the BBj language for the MustangWriter.java class which is part of the mustangproject.org documentation. 


For more information see https://www.mustangproject.org/ and http://github.com/zuGFeRD/mustangproject.


If you intend to use the mustangproject.org library with your BBj project, the following steps will be necessary:

1. add mustang-1.5.1.jar to your classpath

2. create your own implementation similar to MustangWriter.bbj. One way would be to implement database reading and calculation into the disctinct classes (like read item data, invoice header field etc), or you could expose setters for each of the fields needed so you can collect the data from the outside. 

3. use the exporter as follows:

```
mwr! = new MustangWriter()
ze! = new ZUGFeRDExporterFromA1Factory().setProducer("BASIS BBj")
:                    .setCreator(System.getProperty("user.name"))
:                    .load(pdf$)

ze!.PDFattachZugferdFile(mwr!)
ze!.export(pdf_out$)
```

REM TODO: we could create a generic class with generic setters for all of the fields to collect the data from outside, e.g. during an invoicing run.


## create own XML 

(The mustangproject.org library allows adding own XML. Sample needed.)

## The German "Leitweg ID"

German official authorities issue a so called Leitweg ID that has to be transmitted in the Buyer Reference field. 

The sources below seem to recommend that you will need a 46-character long field in the order header to store the Leitweg ID with the order - it looks like there might be different Leitweg IDs possible for the same buyer.

Sources:

https://www.e-rechnung-bund.de/faq-e-rechnung/faq-leitweg-id/

https://ecosio.com/de/blog/was-ist-eine-leitweg-id/#:~:text=Durch%20die%20eindeutige%20Adressierung%20mittels,Verwaltungseinheiten%20und%20den%20darin%20aufgebauten

