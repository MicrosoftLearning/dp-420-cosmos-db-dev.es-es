---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

Este repositorio contiene los ejercicios de laboratorio práctico para el curso de Microsoft [DP-420 Diseño e implementación de aplicaciones nativas en la nube mediante Microsoft Azure Cosmos DB][course-description] y los [módulos autodirigidos en Microsoft Learn][learn-collection] equivalentes. Los ejercicios están pensados para complementar los materiales de aprendizaje y permiten practicar el uso de las tecnologías descritas en estos materiales.

> &#128221; Para completar estos ejercicios, necesitará una suscripción a Microsoft Azure. Puede registrarse para obtener una prueba gratuita en [https://azure.microsoft.com][azure].

## Laboratorios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| Módulo | Laboratorio |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
