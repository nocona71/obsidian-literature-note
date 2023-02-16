---
aliases: ["{%- if authors -%}
{%- for author in authors -%}
{{author}}
{%- endfor -%}
{%- endif -%}
{%- if date -%} ({{date | format("YYYY")}})
{%- endif -%} {{title}} {{caseTitle}}"]
---
{#- infer latest annotation Date -#}
{% macro maxAnnotationsDate() %}
   {%- set tempDate = "" -%}
	{%- for a in annotations -%}
		{%- set testDate = a.date | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate > tempDate or tempDate == ""-%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro %}
{# infer earliest annotation date #}
{%- macro minAnnotationsDate() -%}
   {%- set tempDate = "" -%}
	{%- for a in annotations -%}
		{%- set testDate = a.date | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate < tempDate or tempDate == ""-%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro -%}
{# infer latest note date #}
{%- macro maxNotesDate() -%}
   {%- set tempDate = "" -%}
	{%- for n in notes -%}
		{%- set testDate = n.dateModified | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate > tempDate or tempDate == ""-%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro -%}
{#- infer earliest note date -#}
{%- macro minNotesDate() -%}
   {%- set tempDate = "" -%}
	{%- for n in notes -%}
		{%- set testDate = n.dateAdded | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate < tempDate or tempDate == "" -%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro -%}
{# find earliest date of two dates #}
{%- macro minDate(min1, min2) -%}
		{%- if min1 <= min2 -%}
			{{min1}}
		{%- else -%}
		    {{min2}}
		{%- endif -%}
{%- endmacro -%}
{# find latest date of two dates #}
{%- macro maxDate(min1, min2) -%}
		{%- if min1 >= min2 -%}
			{{min1}}
		{%- else -%}
		    {{min2}}
		{%- endif -%}
{%- endmacro -%}

{%- set colorCategories = {
"yellow": "Relevant / Important",
"red": "Disagree",
"green": "Important to me",
"blue": "Question / Understanding / Vocabulary",
"purple": "Reference / Term to lookup later"
}
-%}
{# lookup Zotero colors in annotations with categories #}
{%- macro colorCategoryToName(noteColor) -%}
{%- if colorCategories[noteColor]-%}
{{colorCategories[noteColor]}}
{% else %}
{{colorCategories["yellow"]}}
{%endif%}
{%- endmacro -%}

{%- set calloutHeaders = {
"highlight": "Relevant / Important",
"strike": "Disagree",
"underline": "Important to me",
"image": "Question / Understanding / Vocabulary"
}
-%}
{# lookup callout headers by type of annotation #}
{%- macro calloutHeader(type) -%}
{%- if calloutHeaders[type]-%}
{{calloutHeaders[type]}}
{% else %}
{{Note}}
{%endif%}
{%- endmacro -%}

{#- handle space characters in zotero tags -#}
{%- set space = joiner(' ') -%} 
{%- macro printTags(rawTags) -%}
	{%- if rawTags.length > 0 -%}
		{%- for tag in rawTags -%}
			#zotero/{{ tag.tag | lower | replace(" ","_") }} {{ space() }} 
		{%- endfor -%}
	{%- endif -%}
{%- endmacro %}

{#- handle | characters in zotero strings used in MD -#}
{% macro formatCell(cellText) -%}
{{ cellText | replace("|","❕")}}
{%- endmacro %}

{%- macro formatDate(testDate, dateFormat) -%}
{%- if testDate -%}
{{date | format (dateFormat)}}
{%- endif %}	
{%- endmacro %}

{#- handle | characters in zotero strings used in MD -#}
{# {%- set comma = joiner(', ') -%} 
{%- macro generateCreators(prefix) -%}
{%- for creatorType, creators in creators | groupby("creatorType") -%}
{{prefix}}{{ creatorType }}:: {{ space() }} 
    {%- for creator in creators -%}
        {{ creator.firstName }} {{ creator.lastName }} 
		{%- if not loop.last -%}
		{{comma()}}
		{%- endif -%}
    {%- endfor %}
{% endfor -%}
{%- endmacro -%} #}

{%- set fields = {
"title": title or caseTitle,
"authors": authors,  
"editors": editors,
"directors": directors,
"podcasters": podcasters,
"scriptwriters": scriptwriters,
"first-entry": minDate(minAnnotationsDate(), minNotesDate()),
"last-entry": maxDate(maxAnnotationsDate(), maxNotesDate()),
"online-uri": uri,
"bibliography": bibliography,
"pdf": pdfZoteroLink, 
"year": formatDate(date, "YYYY"),
"date": formatDate(date, "YYYY-MM-DD"),
"extra": extra,
"citekey": citekey,
"pages": numPages,
"running-time": runningTime,
"type": type,
"itemtype": itemType,
"language": language,
"url": url
}
-%}

{#- generate field safely -#}
{%- macro generateField(prefix, f, p) -%}
{%- if p -%}
{{prefix}}{{f}}:: {{p}}
{% endif %}
{%- endmacro -%}

{#- generate fields based on Zotero properties -#}
{%- macro generateFields(prefix) -%}
{%- for field, property in fields -%}
{%- if property.length > 0 -%}
{{ generateField(prefix, field, property) }}
{%- endif -%}
{%- endfor %}
{%- endmacro -%}

{{printTags(tags)}}
> [!info]- Metadata
{{generateFields("> ") -}}
{% if relations -%}
> 
> > [!note]- References:  
> >
> > | title | proxy note | desktopURI |
> > | --- | --- | --- |
{%- for r in relations -%}
> | {{formatCell(r.title)}} | [[@{{r.citekey}}]] | [Zotero Link]({{r.desktopURI}}) |
{% endfor -%}
{%- endif %}

> [!info]+ abstract
>
{{generateField("> ", "abstract", abstractNote)}}


{% persist "annotations" %}
%% ## Annotations %%
{% set newAnnotations = annotations | filterby("date", "dateafter", lastImportDate) -%}
{% if newAnnotations.length > 0 %}
⬇️*Imported (Annotations) on {{importDate | format("YYYY-MM-DD#HH:mm:ss")}}*⬇️
{% for colorCategory, newAnnotations in newAnnotations | groupby("colorCategory") -%}
#### {{colorCategoryToName(colorCategory | lower)}}
{% for annotation in newAnnotations -%}
> [!annotation-{% if annotation.color %}{{annotation.colorCategory | lower}}]{% endif %} {{calloutHeader(annotation.type)}}
{%- if annotation.annotatedText %} 
> {{annotation.annotatedText | nl2br }} (p. [{{annotation.page}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.page}}&annotation={{annotation.id}})){% endif %}{%- if annotation.imageRelativePath %}
> ![[{{annotation.imageRelativePath}}|300]]
{%- endif %}{%- if annotation.ocrText %}
> {{annotation.ocrText | nl2br}}{%- endif %}
{%- if annotation.comment %} 
> {{annotation.comment | nl2br }}{% endif %}
> [Page {{annotation.page}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.page}}) [[{{annotation.date | format("YYYY-MM-DD#HH:mm")}}]]
> {{printTags(annotation.tags)}}

{% endfor %}
{%- endfor -%}
{%- endif -%}

{% set newNotes = notes | filterby("dateModified", "dateafter", lastImportDate) -%}
{% if newNotes.length > 0 %}
⬇️*Imported (Notes) on: {{importDate | format("YYYY-MM-DD#HH:mm:ss")}}*⬇️
{% for note in newNotes %}
### 🟨 Note (modified: {{ note.dateModified | format("YYYY-MM-DD#HH:mm:ss") }})
{{ note.note | replace ("# ","### ") }}
[Link to note]({{ note.uri }})
{{printTags(note.tags)}}
---
{% endfor %}
{% endif -%} 
{% endpersist -%}
