---
layout: post
title: Merging DOCX files natively in Apex.
date: '2021-06-22-T00:00:00.000-07:00'
author: Sean Cuevo
description: Populating merge fields on a Word doc can be done with just Apex
image: /assets/img/merge-fields.png

tags:
- document generation
- apex
---

<figure>
  <img src="{{site.url}}/assets/img/merge-fields.png" alt="HTML code showing merge field syntax"/>
</figure>

When it comes to document generation in native Salesforce, I always assumed that your only options were something really simple like using [Blob.toPDF](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_methods_system_blob.htm) or rendering a Visualforce page as a PDF and saving it as a file. Neither of there were very pragmatic - the Blob method left much to be desired in terms of formatting and the Visualforce approach was too hard to maintain. As such, when someone would ask for document generation I would just point them to an ISV like Conga or Drawloop. Surely working with a Word document wouldn't be possible in Apex, right?

## The DOCX file format.

I always assumed that a Word document was a complex file format, inscrutable to the likes of me. For a long time that was true - the original .doc file extension was a more proprietary format binary-based format. In the early 2000s, in response to open source solutions like Open Office, Microsoft created the DOCX file format that used XML.

In fact, a DOCX file is actually just a zipped file of various xml documents. You can try it right now - go to Google docs, type some stuff in it, click File >Download > Microsoft Doc (.docx). Unzip that file and in the resulting folder you can see the xml files yourself. The file we'll focus on is is named `document.xml`.

## Working with document.xml

I created a Word document that looked something like this, but formatted:

<hr/>
%%title%%

%%header1%%

%%header2%%

%%content%%
<hr/>

I unzipped that file and took a look at the document.xml file, searched from my merge fields, and they were relatively to find. For example, the "content" merge field was displayed like this:

```
<w:t xml:space="preserve">%%contentText%%</w:t>
```

Seeing that, I realized I could just do a find and replace on these merge fields, re-zip these files as a new archive and *tada*, basic document generation. And *tada* indeed it does work like that. Could I do this programmatically?

## Document generation with Apex,
Reading an XML file in Apex is simple enough - just get the blob from a ContentVersion or an Attachment record and turn it into a string. But how do I get that document.xml file out of the docx file?

Unfortunately, there does not appear to be a native way to extract files from a zip file in Apex. Luckily someone has solved this problem for us. Enter [Zippex](https://github.com/pdalcol/Zippex), a native Apex Zip utility for the salesforce.com platform. I installed this into a scratch org, uploaded my .docx file, and started to break down the problem:

* Unzip the docx file
* Get the document.xml file
* Do a find and replace on the merge fields
* Rezip the file and save the changes.

I first started by just hard coding IDs to make this a little easier so I didn't have to worry about querying for ContentDocuments:

```
ContentVersion cv = [
    SELECT VersionData
    FROM ContentVersion
    WHERE ContentDocumentId = 'SOME_CONTENT_DOCUMENT_ID'
    AND IsLatest = true
];

Zippex sampleZip = new Zippex(cv.VersionData);
Blob fileData = sampleZip.getFile('word/document.xml');
String docxml = fileData.toString();

docxml = docxml.replace(
  '%%title%%', 'Merged Title!',
  '%%header1%%', 'Merged Header 1!',
  '%%header2%%', 'Merged Header 2!',
  '%%content%%', 'Merged Content!',
);

Blob mergedData = Blob.valueof(docxml);
sampleZip.addFile('word/document.xml', mergedData, null);

String newTitle = 'merged' + Datetime.now().getTime();
ContentVersion mergedFile = new ContentVersion();
mergedFile.VersionData = sampleZip.getZipArchive();
mergedFile.Title = newTitle;
mergedFile.ContentLocation= 's';
mergedFile.PathOnClient= newTitle+'.docx';
insert mergedFile;

Id conDoc = [SELECT ContentDocumentId FROM ContentVersion WHERE Id =:mergedFile.Id].ContentDocumentId;
ContentDocumentLink cdl = new ContentDocumentLink();
cdl.ContentDocumentId = conDoc;
cdl.LinkedEntityId = 'SOME_PARENT_RECORD_ID';
cdl.ShareType = 'I';
cdl.Visibility = 'AllUsers';
insert cdl;
```

After I ran that, on my parent account record that I had put my docx attachment, I had a new docx file that had the merge fields populated!

<figure>
  <img src="{{site.url}}/assets/img/apex-doc-gen-hulk.jpg" alt="Hulk saying 'Native Apex document generation"/>
</figure>

With a proof of concept I reworked the code a little bit so that it's a little more reusable. [Click here for the github repo]( https://github.com/seanpat09/docx-merger).

With that code you can do something like this to generate documents with merge fields:

```
//This File must be a docx file
Id contentDocumentId = '<REPLACE_WITH_DOC_ID>';
Id parentId = '<REPLACE_WITH_PARENT_ID>';
Map<String,String> mergeFields = new Map<String,String>{
        '%%mergefield%%' =>  'I HAVE BEEN MERGED',
        '%%header1%%' =>  'NEW HEADER 1',
        '%%header2%%' =>  'NEW HEADER 2'
    };
new DocxMerger()
    .mergeDoc(contentDocumentId, parentId, mergeFields);
```

## Too good to be true?

The document that I merged was tiny, but when I ran the code it took a while. The resulting debug log was 18 megabytes, so this seems to be an expensive operation. So I have no idea what would happen if I tried to merge a larger, more complex document. My hope is that the bottleneck is in the unzipping part itself and if that's the case I can potentially try to do this client side using LWC. Of course, however, that would mean that you can't bulkify this process via something like a batch job. 

I'm actually quite surprised I got this far - usually I give up on these experiments but this wasn't actually too hard. I'll need to figure out where this project goes from here. Do I add more features, or does this end up in the pile of forgotten repos in GitHub? Either way, it was a cool experiment!