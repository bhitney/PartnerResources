{% comment %}
    
    Parameters:
        - include.includemethod         all/any - all=all tags must match
        - include.includetags
        - include.includesecondtags
        - include.includethirdtags
        - include.includefourthtags
        - include.showdescription

        - include.sectiontitle          such as "business apps" or "azure"
        - include.sectionlist           such as "app dev|data, analytics, & ai"

    May not be tested or implemented:
        - include.removetags            specifies tags to hide from display on list
        - include.sortfield
        - include.sortorder
        - include.showdate
        - include.showtags
        - include.showlink              show/hide deep link to content, default "true"
        - include.target                specify a target like target="_blank"
        - include.visualstyle           normal / compact / tiny / others
        - include.limit                 limit of number to display (no pagination support)
        - include.includesecondtags
        - include.includethirdtags
        - include.includefourthtags
        - include.removetext            text to remove from the title

    For parameters, values are strings (no hyphens) 
    and delimited with | if needed. Example:
    includetags="modern analytics academy | vignettes"

{% endcomment %}

{% assign sectionTitle = "" %}
{% assign sectionTitle = include.sectiontitle %}

{% assign sectionList = "" | split: ',' %}
{% assign sectionList = include.sectionlist | split:'|' | compact %}

{% assign tagsToInclude = "" | split: ',' %}
{% assign tagsToInclude = include.includetags | split:'|' | compact %}

{% assign removetext = "" %}
{% assign tagsToRemove = "" | split: ',' %}
{% assign secondAssetsToInclude = "" | split: ',' %}
{% assign thirdAssetsToInclude = "" | split: ',' %}
{% assign fourthAssetsToInclude = "" | split: ',' %}
{% assign postLimit = 0 %}
{% assign currentLimit = 0 %}
{% assign sortField = "updated" %}
{% assign sortOrder = "desc" %}
{% assign showDate = "true" %}
{% assign showTags = "true" %}
{% assign showLink = "true" %}
{% assign visualStyle = "normal" %}
{% assign showDescription = "true" %}
{% if include.showdescription == "false" %}
    {% assign showDescription = "false" %}
{% endif %}
{% if include.includesecondtags %}
    {% assign secondAssetsToInclude = include.includesecondtags | split:'|' | compact %}
{% endif %}
{% if include.includethirdtags %}
    {% assign thirdAssetsToInclude = include.includethirdtags | split:'|' | compact %}
{% endif %}
{% if include.includefourthtags %}
    {% assign fourthAssetsToInclude = include.includefourthtags | split:'|' | compact %}
{% endif %}
{% if include.sortfield %}
    {% assign sortField = include.sortfield %}
{% endif %}
{% if include.sortorder == "asc" %}
    {% assign sortOrder = "asc" %}
{% endif %}
{% if include.showdate == "false" %}
    {% assign showDate = "false" %}
{% endif %}
{% if include.visualstyle %}
    {% assign visualStyle = include.visualstyle %}
{% endif %}
{% if include.showtags == "false" %}
    {% assign showTags = "false" %}
{% endif %}
{% if include.showlink == "false" %}
    {% assign showLink = "false" %}
{% endif %}
{% if include.limit %}
    {% assign postLimit = include.limit | plus: 0 %}
{% endif %}
{% if include.removetext %}
    {% assign removetext = include.removetext %}
{% endif %}

{% capture site_tags %}
{% for tag in site.tags %}
    {% if tag %}
        {{ tag | first }}
        {% unless forloop.last %}|{% endunless %}
    {% endif %}
{% endfor %}
{% endcapture %}
{% assign docs_tags = "" %}
{% for doc in site.docs %}
    {% assign ttags = doc.tags | join:'|' | append:'|' %}
    {% assign docs_tags = docs_tags | append:ttags %}
{% endfor %}
{% assign all_tags = site_tags | append:docs_tags %}
{% assign tags_list = all_tags | replace: "||", "|" | split:'|' | uniq | sort | compact %}
{% assign all_docs = site.docs | sort: "title" %}
{% assign filtered_docs = all_docs | where_exp: "doc","doc.tags contains 'learning plan'" %}

{% comment %}
    Loop through sectionList and pull documents matching each section
{% endcomment %}

{% for sectionTag in sectionList %}
    {% assign lowerTag = sectionTag | downcase %}
    {% assign current_docs = "" | split: ',' %}

    {% for doc in filtered_docs %}
    {% unless doc.tags contains "deprecated" %}

        {% if doc.tags contains lowerTag %}
        {% assign current_docs = current_docs | push: doc %}
        {% endif %}

    {% endunless %}
    {% endfor %}

    {% assign current_docs = current_docs | uniq %}
    {% comment %}
    ----------------------------------------------------
        additional filtering and sorting
        todo: check sortField for nulls
    ----------------------------------------------------
    {% endcomment %}
    {% if sortOrder == "asc" %}
        {% assign current_docs = current_docs | sort: sortField %}
    {% else %}
        {% assign current_docs = current_docs | sort: sortField | reverse %}
    {% endif %}

    {% comment %}
    ----------------------------------------------------
        display results
    ----------------------------------------------------
    {% endcomment %}

    {% if postLimit == 0 %}
        {% assign currentLimit = current_docs.size %}
    {% else %}
        {% assign currentLimit = postLimit %}
    {% endif %}

    {% comment %}
    ----------------------------------------------------
        display header
    ----------------------------------------------------
    {% endcomment %}
<h2>{{ sectionTag }}</h2>

    {% for doc in current_docs limit: currentLimit %}

        {% assign uniquedoctags = doc.tags | uniq | sort %}

        {% assign doctitle = doc.title %} 
        {% if removetext.size > 0 %}
            {% assign doctitle = doc.title | remove: removetext %}
        {% endif %}

        {% if visualStyle == "compact" %}

<div class="tag-entry" style="padding-left:25px;">
    <div><a href="{{- site.baseurl -}}{{- doc.url -}}">{{ doc.title }}</a>
    {% if doc.updated %}
    <span class="docupdated" style="padding-left: 5px;">Updated <time datetime="{{- doc.updated | date_to_xmlschema -}}"> {{- doc.updated | date: "%B %d, %Y" -}}</time></span>
    {% endif %}
    </div>
    {% if showDescription == "true" %}
    <div>{{ doc.description }}</div>
    {% endif %}
</div>
<div style="padding-bottom: 20px;"></div>

        {% elsif visualStyle == "tiny" %}

<div class="tag-entry" style="scroll-margin-top: 5rem;" id="{{ doc.title }}">
    <div>
        {% if showLink == "true" %}
            <a class="nav-entry" href="{{- site.baseurl -}}{{- doc.url -}}">{{ doctitle }}</a> 
        {% else %}
            <span class="nav-entry">{{ doctitle }}</span> 
        {% endif %}
        {% if doc.updated and showDate == "true" %}
            <span class="docupdated"><time datetime="{{- doc.updated | date_to_xmlschema -}}"> {{- doc.updated | date: "%B %d, %Y" -}}</time></span>
        {% endif %}
    </div>
</div>
<div style="clear:both; padding-top: 5px; padding-bottom: 0px;">
</div>

        {% elsif visualStyle == "homepage" %}

<p style="line-height:100%">
<a class="homepagecontent" href="{{- site.baseurl -}}{{- doc.url -}}">{{ doctitle }}</a>
<span class="docupdated"><time datetime="{{- doc.updated | date_to_xmlschema -}}"> {{- doc.updated | date: "%B %d, %Y" -}}</time></span>
</p>

        {% else %}

        {% comment %}
            Assume the visualstyle is "normal" if not matching any other
        {% endcomment %}

<div class="tag-entry">
    <div><a href="{{- site.baseurl -}}{{- doc.url -}}">{{ doc.title }}</a></div>
    {% if showTags == "true" %}
    <div>{% for tag in uniquedoctags %}<span style="font-size:12px" class="badge badge-{{ site.tag_color }}"><a style="cursor:pointer; color:white" href="{% if site.tag_search_endpoint %}{{ site.tag_search_endpoint }}{{ tag }}{% else %}{{ site.url }}{{ site.baseurl }}/tags#{{ tag }} {% endif %}">{{ tag }}</a></span>{% endfor %}</div>
    {% endif %}
    {% if showDescription == "true" %}
    <div>{{ doc.description }}</div>
    {% endif %}
    {% if doc.updated and showDate == "true" %}
    <div class="docupdated">Updated <time datetime="{{- doc.updated | date_to_xmlschema -}}"> {{- doc.updated | date: "%B %d, %Y" -}}</time></div>
    {% endif %}
</div>
<div style="padding-bottom: 20px;"></div>

        {% endif %}
    {% endfor %}
{% endfor %}
