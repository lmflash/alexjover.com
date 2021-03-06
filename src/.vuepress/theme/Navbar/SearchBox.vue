<template>
  <div class="search-box">
    <input
      @input="query = $event.target.value"
      aria-label="Search"
      :value="query"
      autocomplete="off"
      spellcheck="false"
      @focus="focused = true"
      @blur="focused = false"
      @keyup.enter="go(focusIndex)"
      @keyup.up="onUp"
      @keyup.down="onDown">
    <ul class="suggestions"
      v-if="showSuggestions"
      :class="{ 'align-right': alignRight }"
      @mouseleave="unfocus">
      <li class="suggestion" v-for="(s, i) in suggestions"
        :class="{ focused: i === focusIndex }"
        @mousedown="go(i)"
        @mouseenter="focus(i)">
        <a :href="s.path" @click.prevent>
          <span class="page-title">{{ s.title || s.path }}</span>
          <span v-if="s.header" class="header">&gt; {{ s.header.title }}</span>
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
const matches = (item, query) =>
  item.title && item.title.toLowerCase().indexOf(query) > -1;

export default {
  data() {
    return {
      query: "",
      focused: false,
      focusIndex: 0
    };
  },
  computed: {
    showSuggestions() {
      return this.focused && this.suggestions && this.suggestions.length;
    },
    suggestions() {
      const query = this.query.trim().toLowerCase();
      if (!query) {
        return;
      }

      const { pages, themeConfig } = this.$site;
      const max = themeConfig.searchMaxSuggestions || 5;
      const localePath = this.$localePath;
      const res = [];
      for (let i = 0; i < pages.length; i++) {
        if (res.length >= max) break;
        const p = pages[i];
        // filter out results that do not match current locale
        if (this.getPageLocalePath(p) !== localePath) {
          continue;
        }
        if (matches(p, query)) {
          res.push(p);
        } else if (p.headers) {
          for (let j = 0; j < p.headers.length; j++) {
            if (res.length >= max) break;
            const h = p.headers[j];
            if (matches(h, query)) {
              res.push(
                Object.assign({}, p, {
                  path: p.path + "#" + h.slug,
                  header: h
                })
              );
            }
          }
        }
      }
      return res;
    },
    // make suggestions align right when there are not enough items
    alignRight() {
      const navCount = (this.$site.themeConfig.nav || []).length;
      return navCount <= 2;
    }
  },
  methods: {
    getPageLocalePath(page) {
      for (const localePath in this.$site.locales || {}) {
        if (localePath !== "/" && page.path.indexOf(localePath) === 0) {
          return localePath;
        }
      }
      return "/";
    },
    onUp() {
      if (this.showSuggestions) {
        if (this.focusIndex > 0) {
          this.focusIndex--;
        } else {
          this.focusIndex = this.suggestions.length - 1;
        }
      }
    },
    onDown() {
      if (this.showSuggestions) {
        if (this.focusIndex < this.suggestions.length - 1) {
          this.focusIndex++;
        } else {
          this.focusIndex = 0;
        }
      }
    },
    go(i) {
      if (!this.showSuggestions) {
        return;
      }
      this.$router.push(this.suggestions[i].path);
      this.query = "";
      this.focusIndex = 0;
    },
    focus(i) {
      this.focusIndex = i;
    },
    unfocus() {
      this.focusIndex = -1;
    }
  }
};
</script>

<style lang="scss">
@import "~styles/theme.scss";

.search-box {
  display: inline-block;
  position: relative;
  margin-right: $navbar-link-padding-h;

  input {
    cursor: text;
    width: 15rem;
    color: lighten($text-color, 10%);
    display: inline-block;
    border: 1px solid darken($border-color, 10%);
    border-radius: 3rem;
    font-size: 1.5rem;
    line-height: 2.8rem;
    padding: 0 0.5rem 0 2.8rem;
    outline: none;
    transition: all 0.2s ease;
    background: #fff url(./search.svg) 0.8rem 0.7rem no-repeat;
    background-size: 1.4rem;
    &:focus {
      cursor: auto;
      border-color: $accent-color;
    }
  }
  .suggestions {
    background: #fff;
    width: 35rem;
    position: absolute;
    top: 1.5rem;
    border: 1px solid darken($border-color, 10%);
    border-radius: 6px;
    padding: 0.4rem;
    list-style-type: none;
    &.align-right {
      right: 0;
    }
  }
  .suggestion {
    line-height: 1.4;
    padding: 0.4rem 0.6rem;
    border-radius: 4px;
    cursor: pointer;

    a {
      color: lighten($text-color, 35%);

      .page-title {
        font-size: 1.6rem;
        font-weight: 600;
      }
      .header {
        font-size: 1.5rem;
        margin-left: 0.25em;
      }
    }
    &.focused {
      background-color: #f3f4f5;
      a {
        color: $accent-color;
      }
    }
  }
}

@media (max-width: $mq-sm-max) {
  .search-box {
    input {
      cursor: pointer;
      width: 0;
      border-color: transparent;
      position: relative;
      &:focus {
        cursor: text;
        width: 10em;
      }
    }
  }
}

@media (max-width: $mq-sm) and (min-width: $mq-sm-max) {
  .search-box {
    .suggestions {
      left: 0;
    }
  }
}

@media (max-width: $mq-sm) {
  .search-box {
    margin-right: 0;
    .suggestions {
      right: 0;
    }
  }
}

@media (max-width: $mq-xxs-max) {
  .search-box {
    .suggestions {
      width: calc(100vw - 4rem);
    }
    input:focus {
      width: 8rem;
    }
  }
}
</style>
