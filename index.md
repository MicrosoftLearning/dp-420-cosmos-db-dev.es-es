---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

Este repositorio contiene los ejercicios de laboratorio práctico para el curso de Microsoft [DP-420 Diseño e implementación de aplicaciones nativas en la nube mediante Microsoft Azure Cosmos DB][course-description] y los [módulos autodirigidos en Microsoft Learn][learn-collection] equivalentes. Los ejercicios están pensados para complementar los materiales de aprendizaje y permiten practicar el uso de las tecnologías descritas en estos materiales.

> &#128221; Para completar estos ejercicios, necesitarás una suscripción a Microsoft Azure. Puedes registrarte para obtener una prueba gratuita en [https://azure.microsoft.com][azure].

# Laboratorios

## Laboratorios de C#

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign csharp_setup_labs = "" | split: "," %} {% assign csharp_regular_labs = "" | split: "," %} {% assign genai_setup_labs = "" | split: "," %} {% assign genai_python_labs = "" | split: "," %} {% assign genai_javascript_labs = "" | split: "," %}

{% for activity in labs %} {% assign segments = activity.url | split: "/" %}

  {% if segments[1] == "instructions" and segments.size == 3 %} {% if activity.lab.module contains "Setup" %} {% assign csharp_setup_labs = csharp_setup_labs | push: activity %} {% else %} {% assign csharp_regular_labs = csharp_regular_labs | push: activity %} {% endif %}
  
  {% elsif activity.url contains '/gen-ai/python/instructions' %} {% assign genai_python_labs = genai_python_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/javascript/instructions' %} {% assign genai_javascript_labs = genai_javascript_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/common/instructions' and activity.lab.module contains "Setup" %} {% assign genai_setup_labs = genai_setup_labs | push: activity %} {% endif %} {% endfor %}

---

### **Laboratorios de configuración**

| Módulo | Laboratorio |
| --- | --- |
{% for activity in csharp_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratorios**

| Módulo | Laboratorio |
| --- | --- |
{% for activity in csharp_regular_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

## **Laboratorios de IA generativa**

### **Laboratorios de configuración comunes**

| Módulo | Laboratorio |
| --- | --- |
{% for activity in genai_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratorios de JavaScript**

| Módulo | Laboratorio |
| --- | --- |
{% for activity in genai_javascript_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Laboratorios de Python**

| Módulo | Laboratorio |
| --- | --- |
{% for activity in genai_python_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
