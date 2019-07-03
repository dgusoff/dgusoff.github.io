---
layout: post
title: Export to Excel from SharePoint Framework
---

I had an interesting requirement recently to provide a custom Excel data download from a SharePoint page. Although this requirement was for a classic model publishing page, I figured I would blog about it from an SPFX persepective, since that classic app is going to get upgraded to Modern eventually.

## The Requirement

I do a lot of work in the financial services industry, and I can tell you that these folks love Excel! In my case I had built a solution that incorporated a couple or related lists and a document library and the client wanted an easy way to download a copy of that dtat set along with some custom logic. Due to the complexity and nuance of the requested data, the standard "Export to Excel" funcitonality built into SharePOint was not suffiient.

## Third Party tools

It turns out there are a number of third party tools out there, many of them open source and free, that will generate and download Excel files from the browser without any need for back end APIs. I investigated a few of them and decided on Sheet.JS as my library of choice. Sheet.JS is free and open source, it allows a fair bit of power and flexibility in creating files, and it had passable documentation.

## Installing the library

Assuming you've got an SPFX web part created and opened, we install SheetJS in the normal fashion:

````
npm install XLSX
````

Then we can reference the library inside our component:

````
import * as XLSX  from 'xlsx';
````

