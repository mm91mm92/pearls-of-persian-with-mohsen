
**TypeScript:**

```typescript

```

**Jinja2 / Astro template:**

```jinja2
{#
This template uses the following environment and context variables:
environment = {
  labels: Record<string, string>;
}
context = {
  toc: Toc;
}
#}

{% macro render_toc_list(items, prefix='', level=1) %}
  <ul class="ppwm-toc-list" data-level="{{ level }}">
    {% for item in items %}
      {% set number = prefix ~ loop.index %}
      {% set has_children = item.children is defined and item.children %}

      <li class="ppwm-toc-item">
        <div class="ppwm-toc-row">

          <a href="{{ item.url }}" class="ppwm-toc-link">
            <span class="ppwm-toc-number">{{ number }}</span>
            <span class="ppwm-toc-title">{{ item.title }}</span>
          </a>

          {% if has_children %}
            {% set safe_id = 'ppwm-toc-' ~ (number | replace('.', '-')) %}

            <button
              type="button"
              class="ppwm-toc-toggle"
              aria-expanded="true"
              aria-controls="{{ safe_id }}"
              aria-label="تغییر وضعیت بخش"
            >
              <svg
                class="ppwm-toc-caret"
                viewBox="0 0 16 16"
                aria-hidden="true"
              >
                <path
                  d="M4 6l4 4 4-4"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </button>
          {% endif %}

        </div>

        {% if has_children %}
          <div
            class="ppwm-toc-children"
            id="{{ safe_id }}"
          >
            {{ render_toc_list(item.children, number ~ '.', level + 1) }}
          </div>
        {% endif %}

      </li>
    {% endfor %}
  </ul>
{% endmacro %}


<div class="ppwm-toc-block">
  <div class="ppwm-toc-header">
    <span class="ppwm-toc-header-title">فهرست مطالب</span>
  </div>

  {{ render_toc_list(toc) }}
</div>
```

**CSS:** 

```css

```

**JavaScript:** 

```javascript

```