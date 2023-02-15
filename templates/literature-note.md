---
aliases: ["{% if authors %}{% for author in authors %}{{author}}{% endfor %}{% endif %}{% if date %}({{date | format("YYYY")}}){%- endif %}{{title}}{{caseTitle}}"]
{%- if bookTitle %}booktitle: {{bookTitle}} {%- endif %}
year: {% if date %}{{date | format ("YYYY")}}{% endif %}
date: {% if date %}{{date | format ("YYYY-MM-DD")}}{% endif %}
pages: {{pages}}
sourcetype: {{itemType}}
extra: {{extra}}
citekey: {{citekey}}
---
{#- infer latest annotation Date -#}
{%- macro maxAnnotationsDate() -%}
   {%- set tempDate = "" -%}
	{%- for a in annotations -%}
		{%- set testDate = a.dateModified | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate > tempDate or tempDate == ""-%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro -%}
{# infer earliest annotation date #}
{%- macro minAnnotationsDate() -%}
   {%- set tempDate = "" -%}
	{%- for a in annotations -%}
		{%- set testDate = a.dateAdded | format("YYYY-MM-DD#HH:mm:ss") -%}
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
{# lookup Zotero colors in annotations with categories #}
{%- macro colorCategoryToName(colorCategory) -%}
	{%- switch colorCategory -%}
		{%- case "yellow" -%}
			"Relevant / Important"
		{%- case "red" -%}
			"Disagree"
		{%- case "green" -%}
			"Important to me"
		{%- case "blue" -%}
			"Question / Understanding / Vocabulary"
		{%- case "purple" -%}
			"Reference / Term to lookup later"
		{%- default -%}
			"other"
	{%- endswitch -%}
{%- endmacro -%}
{# lookup callout headers by type of annotation #}
{%- macro calloutHeader(type) -%}
	{%- switch type -%}
		{%- case "highlight" -%}
			Highlight
		{%- case "strike" -%}
			Strikethrough
		{%- case "underline" -%}
			Underline
		{%- case "image" -%}
			Image
		{%- default -%}
			Note
	{%- endswitch -%}
{%- endmacro %}
{#- handle space characters in zotero tags -#}
{%- set space = joiner(' ') -%} 
{%- macro printTags(rawTags) -%}
	{%- if rawTags.length > 0 -%}
		{%- for tag in rawTags -%}
			#{{ tag.tag | lower | replace(" ","_") }} {{ space() }} 
		{%- endfor -%}
	{%- endif -%}
{%- endmacro %}
{#- handle | characters in zotero strings used in MD -#}
{%- macro formatCell(cellText) -%}
{{ cellText | replace("|","❕")}}
{%- endmacro %}
{{printTags(tags)}}
> [!info]- Metadata
> title:: "{{title}}{{caseTitle}}"
> abstract::  {{abstractNote}}
> authors: {{authors}}
> editors: {{editors}}
> first-entry:: {{minDate(minAnnotationsDate(), minNotesDate() ) }}
> last-entry:: {{maxDate(maxAnnotationsDate(), maxNotesDate() ) }}
> online-uri:: {{uri}} 
> desktop-uri:: {{desktopURI}}
> bibliography:: {{bibliography}}
> pdf:: {{pdfZoteroLink}}

> [!note]- References:  
> 
> | title | proxy note | desktopURI |
> | --- | --- | --- |
{% if relations.length > 0 -%}
{%- for r in relations -%}
> | {{formatCell(r.title)}} | [[@{{r.citekey}}]] | [Zotero Link]({{r.desktopURI}}) |
{% endfor -%}
{%- endif %}
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

%% ## Notes %%
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
