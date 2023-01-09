---
slug: {{ substr (md5 (printf "%s%s" .Date (replace .TranslationBaseName "-" " " | title))) 4 8 }}
author: "昊色居士"
title: "{{ replace .Name "-" " " | title }}"
description: 
date: {{ .Date }}
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags: [

]
categories: [

]
series:  [

]
---
