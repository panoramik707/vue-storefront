<template>
  <div id="home">
    <main-slider />

    <!-- <promoted-offers collection="smallBanners" :limit="2" :columns="2" class="mt-2 mb-16 sm:my-8" /> -->

    <!-- <section class="new-collection container mb-16">
      <div>
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Shop new arrivals') }}</h2>
        </header>
      </div>
      <div class="row center-xs">
        <product-listing columns="4" :products="newCollection" />
      </div>
    </section> -->

    <section class="new-collection container mb-16" style="margin-top: 100px;" v-if="!!trendingCollection && trendingCollection.length > 0">
      <div>
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Trending') }}</h2>
        </header>
      </div>
      <div class="row center-xs">
        <product-listing columns="4" :products="trendingCollection" />
      </div>
    </section>

    <section class="new-collection container mb-16" v-if="!!popularCollection && popularCollection.length > 0">
      <div>
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Popular Categories') }}</h2>
        </header>
      </div>
      <div class="row center-xs">
        <product-listing columns="4" :products="popularCollection" />
      </div>
    </section>

    <!-- <promoted-offers collection="smallBanners" :limit="2" :offset="2" :columns="2" class="mt-2 mb-16 sm:my-8" /> -->

    <!-- <products-slider class="mb-16" :title="$t('Sale and discount')" :products="salesCollection" :config="sliderConfig" /> -->

    <!-- <section class="container mb-16">
      <div class="justify-center">
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Get inspired') }}</h2>
        </header>
      </div>
      <tile-links />
    </section> -->
    <Onboard/>

  </div>
</template>

<script>
// 3rd party dependecies
import { prepareQuery } from '@vue-storefront/core/modules/catalog/queries/common'

// Core pages
import Home from '@vue-storefront/core/pages/Home'

// Theme core components
import ProductListing from 'theme/components/core/ProductListing'
import ProductsSlider from 'theme/components/core/ProductsSlider'
import MainSlider from 'theme/components/core/blocks/MainSlider/MainSlider'

// Theme local components
import Onboard from 'theme/components/theme/blocks/Home/Onboard'
import PromotedOffers from 'theme/components/theme/blocks/PromotedOffers/PromotedOffers'
import TileLinks from 'theme/components/theme/blocks/TileLinks/TileLinks'
import { Logger } from '@vue-storefront/core/lib/logger'
export default {
  mixins: [Home],
  components: {
    MainSlider,
    Onboard,
    ProductListing,
    ProductsSlider,
    PromotedOffers,
    TileLinks
  },
  data () {
    return {
      sliderConfig: {
        perPage: 1,
        perPageCustom: [[0, 2], [1024, 4]],
        paginationEnabled: true,
        loop: false,
        paginationSize: 6
      }
    }
  },
  computed: {
    categories () {
      return this.$store.state.category.list
    },
    newCollection () {
      return this.$store.state.homepage.new_collection
    },
    salesCollection () {
      return this.$store.state.homepage.sales_collection
    },
    trendingCollection () {
      return this.$store.state.homepage.trending_collection
    },
    popularCollection () {
      return this.$store.state.homepage.popular_collection
    }
  },
  created () {
    // Load personal and shipping details for Checkout page from IndexedDB
    this.$store.dispatch('checkout/load')
  },
  beforeMount () {
    if (this.$store.state.__DEMO_MODE__) {
      this.$store.dispatch('claims/check', { claimCode: 'onboardingAccepted' }).then((onboardingClaim) => {
        if (!onboardingClaim) { // show onboarding info
          this.$bus.$emit('modal-toggle', 'modal-onboard')
          this.$store.dispatch('claims/set', { claimCode: 'onboardingAccepted', value: true })
        }
      })
    }
  },
  asyncData ({ store, route }) { // this is for SSR purposes to prefetch data
    const config = store.state.config
    const concernedCategoriesList = config.concerned_categories_list
    return new Promise((resolve, reject) => {
      Logger.info('Calling asyncData in Home (theme)')()

      let trendingQuery = prepareQuery({ queryConfig: 'trending' })
      let popularQuery = prepareQuery({ queryConfig: 'popular' })

      store.dispatch('category/list', { includeFields: config.entities.optimize ? config.entities.category.includeFields : null }).then((categories) => {
        store.dispatch('product/list', {
          query: popularQuery,
          size: 8,
          sort: 'created_at:desc',
          includeFields: config.entities.optimize ? (config.products.setFirstVarianAsDefaultInURL ? config.entities.productListWithChildren.includeFields : config.entities.productList.includeFields) : []
        }).catch(err => {
          reject(err)
        }).then((res) => {
          if (res) {
            let concernedCategoriesIds = categories.items.filter(category => concernedCategoriesList.includes(category.name)).map(category => category.id)
            let items = []
            res.items.forEach(
              (item) => {
                let categoryId = item.category_ids.find(categoryId => concernedCategoriesIds.includes(parseInt(categoryId)))
                if (!!categoryId && items.find(product => product.id === item.id) === undefined) {
                  item.id = parseInt(categoryId)
                  item.slug = categories.items.find(category => category.id === parseInt(categoryId)).slug
                  item.name = categories.items.find(category => category.id === parseInt(categoryId)).name
                  item.hide_price = true
                  item.is_category = true
                  let category = {}
                  Object.assign(category, item)
                  items.push(category)
                }
              }
            )
            store.state.homepage.popular_collection = items
          }
          return resolve()
        })

        store.dispatch('product/list', {
          query: trendingQuery,
          size: 8,
          sort: 'created_at:desc',
          includeFields: config.entities.optimize ? (config.products.setFirstVarianAsDefaultInURL ? config.entities.productListWithChildren.includeFields : config.entities.productList.includeFields) : []
        }).catch(err => {
          reject(err)
        }).then((res) => {
          if (res) {
            store.state.homepage.trending_collection = res.items
          }
          return resolve()
        })
      }).catch(err => {
        reject(err)
      })
    })
  }
}
</script>
