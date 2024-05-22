{#- infer latest annotation Date -#}
{%- macro maxAnnotationsDate() -%}
   {%- set tempDate = "" -%}
	{%- for a in annotations -%}
		{%- set testDate = a.date | format("YYYY-MM-DD#HH:mm:ss") -%}
		{%- if testDate > tempDate or tempDate == ""-%}
			{%- set tempDate = testDate -%}
		{%- endif -%}
	{%- endfor -%}
	{{tempDate}}
{%- endmacro -%}

{%- set colorCategoryToMeaning = {
"yellow": "Relevant, Important",
"red": "Disagree",
"green": "Important to me",
"blue": "Question, Understanding, Vocabulary",
"purple": "Reference, Term to lookup later",
"magenta": "margenta - undefined",
"orange": "orange - undefined",
"gray": "gray - undefined"
}-%}

{# lookup Zotero colors in annotations with Category #}
{%- macro getMeaning(colorCategory) -%}
	{%- if colorCategory-%}
		{{- colorCategoryToMeaning[colorCategory] -}}
	{%- else -%}
		{{- colorCategoryToMeaning["yellow"] -}}
	{%- endif -%}
{%- endmacro -%}

{%- set calloutHeaders = {
"highlight": "Highlight",
"strike": "Strike Through",
"underline": "Underline",
"note": "Sticky Note",
"image": "Image"
}-%}

{# lookup callout headers by type of annotation #}
{%- macro calloutHeader(type) -%}
	{%- if calloutHeaders[type]-%}
		{{- calloutHeaders[type] -}}
	{%- else -%}
		{{Note}}
	{%- endif -%}
{%- endmacro -%}

{#- handle space characters in zotero tags -#}
{%- macro printTags(rawTags) -%}
	{%- if rawTags.length > 0 -%}
		{%- for tag in rawTags -%}
			#zotero/{{ tag.tag | lower | replace(" ","_") }} {{ ' ' }} 
		{%- endfor %}
	{%- endif %}
{%- endmacro -%}

{%- set inline_fields = {
"abstract": abstractNote,
"pdf": pdfZoteroLink, 
"extra": '"' ~ extra ~ '"',
"bibliography": '"' ~ bibliography ~ '"'
}
-%}

{%- set frontmatter_fields = {
"title": '"' ~ (title | replace ('"','') or caseTitle | replace ('"','')) ~ '"',
"authors": '[' ~ authors | replace (";", ", ") ~ ']',
"editors": '[' ~ editors | replace (";", ", ") ~ ']',
"directors": '[' ~ directors | replace (";", ", ") ~ ']',
"podcasters": '[' ~ podcasters | replace (";", ", ") ~ ']',
"scriptwriters": '[' ~ scriptwriters | replace (";", ", ") ~ ']',
"first-entry": minAnnotationsDate,
"last-entry": maxAnnotationsDate,
"online-uri": uri,
"year": date | format("YYYY"),
"date": date | format("YYYY-MM-DD"),
"citekey": citekey,
"pages": numPages,
"running-time": runningTime,
"type": type,
"class": itemType,
"language": language,
"url": url,
"isbn": ISBN,
"cover": "https://covers.openlibrary.org/b/isbn/"+ISBN | replace ("-","")+"-M.jpg"
}
-%}

{# generate field safely -#}
{%- macro generateField(prefix, delimiter, f, p) -%}
{%- if p and p != "[undefined]"-%}
{{prefix}}{{f}}{{delimiter}}{{p}}
{% endif %}
{%- endmacro -%}

{#- generate fields based on Zotero properties -#}
{%- macro generateFields(prefix, delimiter, fields) -%}
{%- for field, property in fields -%}
{%- if property.length > 0 -%}
{{- generateField(prefix, delimiter, field, property) -}}
{%- endif -%}
{%- endfor -%}
{%- endmacro -%}

---
aliases: ["{{title | replace ('"','')}}"{%- if authors and date-%}, "
{%- for author in authors -%}
{{author}}
{%- endfor -%}
{{" ("+date | format("YYYY") +") "}}{{title | replace ('"','')}}{{caseTitle | replace ('"','')}}"{%- endif -%}]
{{generateFields("",": ",frontmatter_fields) -}}
---

{%- if ISBN -%}
![|200](https://covers.openlibrary.org/b/isbn/{{ISBN | replace ("-","")}}-M.jpg)
{%- endif -%}
{{ "" }}

{{printTags(tags)}}

> [!info]- Metadata
{{generateFields("> ",":: ",inline_fields) -}}
{% if relations.length > 0 -%}
> 
> > [!note]- References:  
> >
> > | title | proxy note | desktopURI |
> > | --- | --- | --- |
{%- for r in relations %}
> > | {{r.title | replace("|","❕")}} | [[@{{r.citekey}}]] | [Zotero Link]({{r.desktopURI}}) |
{%- endfor -%}
{{ "" }}
{%- endif %}
{{ "" }}
🔥🔥🔥everything above this line might change during an update 🔥🔥🔥
{% persist "notes" %}
{%- set newNotes = notes | filterby("dateModified", "dateafter", lastImportDate) -%}
{% if newNotes.length > 0 %}
⬇️*Imported (Notes) on: {{importDate | format("YYYY-MM-DD#HH:mm:ss")}}*⬇️
{% for note in newNotes %}
### 🟨 Note (modified: {{ note.dateModified | format("YYYY-MM-DD#HH:mm:ss") }})
{{ "" }}
{#- change heading level -#}
{{- note.note | replace ("# ","### ") -}}
{{ "" }}
[Link to note](zotero://select/library/items/{{note.key}})
{{printTags(note.tags)}}

---
{%- endfor %}
{% endif -%} 

{% endpersist -%}
{{ " " }}
{% persist "annotations" %}

{%- set newAnnotations = annotations | filterby("date", "dateafter", lastImportDate) -%}
{% if newAnnotations.length > 0 %}
⬇️*Imported (Annotations) on {{importDate | format("YYYY-MM-DD#HH:mm:ss")}}*⬇️
{% for annotation in newAnnotations %}
> [!annotation-{{ annotation.colorCategory | lower}}] {{getMeaning(annotation.colorCategory | lower)}}
> {%- if annotation.tags.length > 0 %} 
> {{printTags(annotation.tags)}}
{%- endif %}
{%- if annotation.annotatedText.length > 0 %} 
> 
> {{-annotation.annotatedText | nl2br -}} 
{%- endif %}
{%- if annotation.imageRelativePath %}
> ![[{{annotation.imageRelativePath}}|300]]
{%- endif %}
{%- if annotation.comment %} 
> 
> **comment:**
> {{annotation.comment | nl2br }}
{%- endif %}
> 
> {{annotation.date | format("YYYY-MM-DD#HH:mm")}}
{%- if annotation.desktopURI %} 
> 
> (see [PDF p. {{annotation.page}}]({{annotation.desktopURI}}))
{%- endif %}
{% endfor %}
{# {% endfor %} #}
{%- endif -%}
{%- endpersist -%}