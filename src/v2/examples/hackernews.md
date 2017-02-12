---
title: Clon de HackerNews
type: ejemplos
order: 10
---

> Este es un clo de HackerNews construido con la API oficial de HN en Firebase, Vue 2.0 + vue-router + vuex, con renderizado del lado servidor.

{% raw %}
<div style="max-width:600px">
  <a href="https://github.com/vuejs/vue-hackernews-2.0" target="_blank">
    <img style="width:100%" src="/images/hn.png">
  </a>
</div>
{% endraw %}

> [Demo en vivo](https://vue-hn.now.sh/)
> Nota: la demostración puede necesitar algo de tiempo en inicializar si hace tiempo que nadie ha accedido.
>
> [[Fuente](https://github.com/vuejs/vue-hackernews-2.0)]

## Características

- Renderizado del lado servidor
  - Vue + vue-router + vuex trabajando juntos
  - Pre carga de datos del lado servidor
  - Estado en el lado cliente & regeneración de DOM
- Componentes de vue de un solo archivo
  - Recarga en caliente durante el desarrollo
  - Extracción de CSS para producción
- Actualizaciones en tiempo real de las listas con animaciones FLIP

## Vistazo general de la arquitectura

<img width="973" alt="Hackernew clone architecture overview" src="/images/hn-architecture.png">
