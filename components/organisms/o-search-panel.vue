<template>
  <div class="o-search-panel">
    <div v-show="isShowLoader" class="search-result-loader"></div><div
      v-if="noResultsMessage"
      class="no-results"
    >
      {{ $t(noResultsMessage) }}
    </div>
    <div v-else class="container">
      <div class="categories">
        <h1 class="categories__title">
          {{ $t("Categories") }}
        </h1>
        <SfList v-if="visibleProducts.length && categories.length > 1" class="categories__listing">
          <SfListItem
            v-for="category in categories"
            :key="category.category_id"
          >
            <SfMenuItem
              :class="{'selected': isCategorySelected(category)}"
              :label="category.name"
              icon=""
              @click="toggleCategory(category)"
            />
          </SfListItem>
        </SfList>
      </div>
      <div class="products">
        <h1 class="products__title">
          {{ $t("Product suggestions") }}
        </h1>
        <div class="products__listing">
          <SfProductCard
            v-for="product in visibleProducts"
            :key="product.id"
            :title="product.title"
            :image="product.image"
            :imageWidth="150"
            :imageHeight="150"
            :link="product.link"
            link-tag="router-link"
            :wishlist-icon="false"
            class="products__product-card"
            @click.native="$store.commit('ui/setSearchpanel', false)"
          >
            <template slot="title">
              <h3 class="sf-product-card__title">
                {{ product.title }}
              </h3>
              <p class="reference">
                  Ref : {{ product.sku }}
              </p>
              <div class="d-flex align-start justify-between">
                  <SfPrice
                    class="sf-product-card__price"
                    :regular="product.price.regular"
                    :special="product.price.special" 
                  />    
              </div>
               <SfLink class="sf-button a-add-to-cart sf-button--full-width sf-add-to-cart__button" :link="product.link">
                 More Info
              </SfLink> 
            </template> 
          </SfProductCard>
        </div>
        <div v-show="isShowLoader" class="load-more-loader"></div>
        <SfButton
          v-if="OnlineOnly && readMore && visibleProducts.length >= pageSize"
          class="sf-button--full-width load-more"
          type="button"
          @click="$emit('see-more')"
        >
          {{ $t("Load more") }}
        </SfButton>
      </div>
    </div>
  </div>
</template>

<script>
import { prepareCategoryProductSearch  } from 'theme/helpers';
import VueOfflineMixin from 'vue-offline/mixin';
import { SfButton, SfList, SfMenuItem, SfProductCard, SfPrice, SfLink } from '@storefront-ui/vue';
import { disableBodyScroll, clearAllBodyScrollLocks } from 'body-scroll-lock';
import { mapState } from 'vuex';

export default {
  name: 'OSearchPanel',
  components: {
    SfButton,
    SfList,
    SfMenuItem,
    SfProductCard,
    SfPrice,
    SfLink
  },
  mixins: [VueOfflineMixin],
  data () {
    return {
      selectedCategoryIds: []
    };
  },
  props: {
    search: {
      type: String,
      required: true,
      default: ''
    },
    pageSize: {
      type: Number,
      required: true,
      default: 16
    },
    products: {
      type: Array,
      required: true,
      default: () => ([])
    },
    readMore: {
      type: Boolean,
      required: true,
      default: false
    }
  },
  computed: {
     ...mapState({
      isShowLoader: state => state.ui.customLoader
    }),
    visibleProducts () {
      const productList = this.selectedCategoryIds.length
        ? this.products.filter(product => product.category_ids.some(categoryId => this.selectedCategoryIds.includes(categoryId)))
        : this.products;

      return productList.map(product => prepareCategoryProductSearch(product));
    },
    categories () {
      const distinctCategories = this.products
        .filter(product => product.category)
        .map(product => product.category)
        .flat()
        .reduce((result, category) => {
          result[category.category_id] = category;
          return result;
        }, {});

      return Object.values(distinctCategories);
    },
    noResultsMessage () {
      return this.search.length < 3
        ? 'Searched term should consist of at least 3 characters.'
        : !this.visibleProducts.length
          ? 'No results were found.'
          : '';
    }
  },
  methods: {
    isCategorySelected (category) {
      return this.selectedCategoryIds.includes(category.category_id);
    },
    toggleCategory (category) {
      if (this.isCategorySelected(category)) {
        this.selectedCategoryIds = this.selectedCategoryIds.filter(categoryId => categoryId !== category.category_id);
      } else {
        this.selectedCategoryIds.push(category.category_id);
      }
    }
  },
  watch: {
    categories () {
      this.selectedCategoryIds = [];
    }
  },
  mounted () {
    disableBodyScroll(this.$el)
  },
  destroyed () {
    clearAllBodyScrollLocks()
  }
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.o-search-panel {
  position: fixed;
  left: 0;
  right: 0;
  top: var(--_header-height);
  background: var(--c-white);
  overflow: auto;
  max-height: calc(66vh - var(--_header-height));

  @include for-mobile {
    top: auto;
    max-height: calc(100vh - var(--_header-height) - var(--bottom-navigation-height));
  }

  .container {
    display: flex;
    padding-left: 40px;
    padding-right: 40px;
    max-width: 1272px;
    margin: auto;
    @include for-desktop {
      border-top: 1px solid var(--c-light);
    }
    @include for-mobile {
      flex-direction: column;
    }
  }

  .categories {
    @include for-desktop {
      flex: 0 0 20%;
      padding-right: 3rem;
      border-right: 1px solid var(--c-light);
    }

    &__title {
      padding: 0;
      font-size: var(--font-lg);
      font-weight: 500;
      line-height: 3;
    }
    &__listing {
      @include for-desktop {
        margin-top: 2rem;
      }

      .sf-list__item {
        padding: 0.3rem 0;
      }
      .sf-menu-item.selected {
        --menu-item-font-weight: 500;
        text-decoration: underline;
      }
    }
  }

  .products {
    width: 100%;
    &__title {
      padding: 0;
      font-size: var(--font-lg);
      font-weight: 500;
      line-height: 3;
    }
    &__listing {
      display: flex;
      flex: 0 1 auto;
      flex-wrap: wrap;
      margin: -1rem -1rem 0;
    }
    &__product-card {
      flex: 0 1 25%;
      min-width: calc(var(--product-card-max-width) * 0.8);
    }

    @include for-desktop {
      padding-left: 3rem;
    }
  }

  .no-results {
    height: 5rem;
    line-height: 5rem;
    display: flex;
    justify-content: center;
  }

  .load-more {
    margin: var(--spacer-xl) 0;
  }
}
</style>
