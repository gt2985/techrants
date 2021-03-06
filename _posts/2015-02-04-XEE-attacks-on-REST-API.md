---
title: "Secure APIs against XEE Attacks(XML Injection Attacks)"
categories:
  - Technology
  - Java
tags:
  - Security
  - Input Sanitization
  - XML Expansion
  - XEE Attack
---
Recently while testing my REST API for invalid XML data, I found an XEE susceptibility. 

XEE attacks are possible as we send xml payloads which get parsed by the parsers at runtime. Major XEE attacks can be clasified as 
1. XML Injection Attacks
2. XML Expansion Attacks

XML Injection attacks can be made using external url or schema file references in the xml payload. One can point to an url that never returns, turning the request into a kind of DOS attack. Similarly by referencing an external file it is also possible to gain access to the files present on the server, if the file system access is not properly implemented on the same. 

XML Expansion attacks, are made by using doctype references in the xml payload, which are recursive or just huge. LOL attack is a famous example of the same. 
An example payload can be seen as below,
<pre><code>
&lt;?xml version="1.0" encoding="UTF-8" standalone="yes"?&gt;
&lt;!DOCTYPE foo [ &lt;!ENTITY a "1234567890" &gt; &lt;!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;" &gt; &lt;!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;" &gt; &lt;!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;&c;&c;" &gt; &lt;!ENTITY e "&d;&d;&d;&d;&d;&d;&d;&d;&d;&d;" &gt; &lt;!ENTITY f "&e;&e;&e;&e;&e;&e;&e;&e;&e;&e;" &gt; &lt;!ENTITY g "&f;&f;&f;&f;&f;&f;&f;&f;&f;&f;" &gt; &lt;!ENTITY h "&g;&g;&g;&g;&g;&g;&g;&g;&g;&g;" &gt; &lt;!ENTITY i "&h;&h;&h;&h;&h;&h;&h;&h;&h;&h;" &gt; &lt;!ENTITY j "&i;&i;&i;&i;&i;&i;&i;&i;&i;&i;" &gt; ]&gt; 
&lt;sometag xmlns="somensreference"&gt;
&lt;data&gt;&j;&lt;/data&gt;
&lt;/sometag&gt;
</code></pre>

I found it quite peculiar to ensure that APIs are secure from these attacks. 

When using RestEasy, the easiest way to do this was to write a custom rest easy content provider, implementing
<CODE>javax.ws.rs.ext.MessageBodyReader&lt;Object&gt; </CODE> interface.

Then ensuring that the unmarshaller instance that gets provided to the contentprovider, has the following flags set on it. 

<pre><code>
javax.xml.parsers.SAXParserFactory factory = SAXParserFactory.newInstance();
factory.setFeature("http://xml.org/sax/features/validation", false);
factory.setFeature("http://xml.org/sax/features/namespaces", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
</code></pre>

We can use the factory created above to create an XMLReader and use it to unmarshal the xml. 

<pre><code>
org.xml.sax.XMLReader reader = factory.newSAXParser().getXMLReader();
//Get the unmarshaller here
javax.xml.bind.Unmarshaller unmarshaller = getUnMarshaller();
//source- variable which get you a reference to the input xml, payload
unmarshaller.unmarshal(new SAXSource(createXmlReader(), source));
</code></pre>

Hope that helps us secure our APIs from unwanted XML attacks.
