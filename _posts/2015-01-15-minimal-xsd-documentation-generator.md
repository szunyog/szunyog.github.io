---
layout: post
title:  "Minimal XSD documentation generator"
date:   2015-01-15 20:31:49
categories: xml
tags: [xml, xsd]
description: Very simple documentation generator from an XSD file.
---

This is a simple XSLT to produce a HTML documentation of an XSD file.

{% highlight xml %}
<?xml version="1.0"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                xmlns:xsi="http://www.w3.org/2001/10/XMLSchema-instance"
                version="1.0">
  <xsl:output method="html"/>
  <xsl:template match="/">
    <HTML>
      <BODY>
        <xsl:apply-templates select="//xsd:complexType[@name]">
          <xsl:sort select="@name" />
        </xsl:apply-templates>
        <xsl:apply-templates select="//xsd:simpleType[@name]">
          <xsl:sort select="@name" />
        </xsl:apply-templates>
      </BODY>
    </HTML>
  </xsl:template>
  <xsl:template match="xsd:complexType[@name]">
    <h4>
      <xsl:value-of select="@name"/>
    </h4>
    <table border="1" width="600px">
      <tr>
        <th width="40%">Name</th>
        <th width="40%">Type</th>
        <th width="20%">Occurance</th>
      </tr>
      <xsl:apply-templates/>
    </table>
  </xsl:template>
  <xsl:template match="xsd:simpleType[@name]">
    <h4>
      <xsl:value-of select="@name"/>
    </h4>
    <xsl:apply-templates/>
  </xsl:template>
  <xsl:template match="xsd:restriction[@base]">
    <ul>
      <xsl:for-each select="xsd:enumeration">
        <li>
          <xsl:value-of select="@value"/>
        </li>
      </xsl:for-each>
    </ul>
  </xsl:template>
  <xsl:template match="xsd:element">
    <xsl:variable name="name" select="@name"/>
    <xsl:variable name="type" select="@type"/>
    <tr>
      <td>
        <xsl:value-of select="$name"/>
      </td>
      <td>
        <xsl:value-of select="$type"/>
      </td>
      <td>
        <xsl:choose>
          <xsl:when test="not(@maxOccurs)">
            <xsl:value-of select="@minOccurs"/>-1
          </xsl:when>
          <xsl:when test="@maxOccurs=@minOccurs">
            <xsl:value-of select="@minOccurs"/>
          </xsl:when>
          <xsl:when test="@maxOccurs='unbounded'">
            <xsl:value-of select="@minOccurs"/>-*
          </xsl:when>
          <xsl:otherwise>
            <xsl:value-of select="@minOccurs"/>-<xsl:value-of select="@maxOccurs"/>
          </xsl:otherwise>
        </xsl:choose>
      </td>
    </tr>
  </xsl:template>
  </xsl:stylesheet>
{% endhighlight %}

