# Links internos disparam refresh completo (fora do fluxo SPA)

- **Data:** 2026-09-03
- **Tags:** inertia, vue, spa, navegacao, primevue, ziggy
- **Status:** resolvido

## Contexto

Laravel 13 + Inertia 3 + Vue 3.5 + PrimeVue 4.5 (styled mode) + Ziggy. Ocorre ao
implementar módulos novos a partir deste template: telas de listagem, botões "Novo",
links de "ver detalhes", menus de ação, breadcrumbs de páginas novas.

## Sintoma

Ao clicar num link para outra rota da própria aplicação, o navegador faz um
carregamento completo da página (flash branco, perde o estado da SPA, refaz o boot
do Vue/Inertia, recarrega CSS/JS) em vez de uma navegação Inertia (troca de
componente sem reload).

## Causa raiz

Navegação nativa do navegador, por fora do Inertia. Dois padrões que causam isso:

1. `<a :href="...">` puro (ou com `route('...')`) apontando para rota interna. Uma
   âncora comum sempre faz request HTTP completo — o Inertia só intercepta o
   componente `<Link>`.
2. Componentes PrimeVue com _model_ de itens (`Menu`, `Breadcrumb`, `PanelMenu`,
   `TieredMenu`, `Menubar`) configurados apenas com `url` no item. O PrimeVue
   renderiza um `<a href>` real para itens com `url`, também fora do Inertia.

## Solução

Padrão do projeto — nunca navegar para rota interna com âncora nativa:

- **Link declarativo:** Inertia `<Link :href="route('...')">` de `@inertiajs/vue3`.
  Referência: `resources/js/components/AppSidebar.vue` (logo + itens de navegação).

- **Navegação imperativa** (dentro de handler, após uma ação/submit):
  `router.visit(route('...'))` / `router.post(...)` de `@inertiajs/vue3`.
  Referência: `resources/js/components/UserMenu.vue` — itens de menu navegam pelo
  callback `command`, não por `url`.

- **Componentes PrimeVue de menu:** não usar `url` no item. Ou:
    - definir `command: () => router.visit(route('...'))` no item; ou
    - sobrescrever o slot `#item` e renderizar `<Link>` ou
      `<a :href="..." @click.prevent="router.visit(url)">`.
      Referência: `resources/js/components/Breadcrumbs.vue` — slot `#item` + função
      `navigate()` que faz `event.preventDefault()` e `router.visit(url)`.

- **`route()`** sempre importado de `ziggy-js`; nunca montar path na mão.

- `<a href>` para destino **externo** (outro site, download de arquivo) é correto e
  deve continuar âncora nativa — a regra vale só para rotas da aplicação.

## Como evitar ou detectar antes

- Ao criar página/módulo: todo alvo de rota interna é `<Link>` ou `router.*`.
- `grep -rn '<a :href\|<a href' resources/js/pages resources/js/components` — cada
  ocorrência para rota interna deve ter `@click` com `router.visit`, senão está
  quebrando o SPA.
- Item de menu PrimeVue com `url:` é sinal de alerta — trocar por `command`.
- Teste manual: clicar no link não pode causar flash de reload nem spinner de
  carregamento completo na aba.

## Referências

- `resources/js/components/AppSidebar.vue`
- `resources/js/components/Breadcrumbs.vue`
- `resources/js/components/UserMenu.vue`
- https://inertiajs.com/links
