---
seo:
  title: Api responder for Laravel
  description: A simple api response template using via DI
---

::u-page-hero
---
orientation: horizontal
---
  :::prose-pre{filename="Terminal"}
  ```bash
  composer require pepperfm/api-responder-for-laravel
  ```
  :::

#title
Standardize your API Responses without traits

#description
API Responder is a class of simple methods that allows you to easily standardize your JSON responses for REST and beyond with minimal effort.

#links
  :::u-button{size="xl" to="/docs" trailing-icon="i-lucide-arrow-right"}
  Get started
  :::
::

::u-page-section
#title
Simplicity is beautiful

#links
  :::u-button
  ---
  color: neutral
  size: lg
  target: _blank
  to: /docs
  trailingIcon: i-lucide-arrow-right
  variant: subtle
  ---
  Documentation
  :::

#features
  :::u-page-feature
  ---
  icon: i-heroicons:squares-2x2
  ---
  #title
  Minimal effort required from the developer
  
  #description
  Simply retrieve the object from the container and return the resulting instance as a JSON response
  :::

  :::u-page-feature
  ---
  icon: i-heroicons:code-bracket-square
  ---
  #title
  Easy to use
  
  #description
  Just pass an array or an Arrayable parameter
  :::

  :::u-page-feature
  ---
  icon: i-heroicons:rocket-launch-16-solid
  ---
  #title
  Readability
  
  #description
  Methods are designed with REST context in mind
  :::
::
