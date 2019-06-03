<template>
  <div id="home">
    <main-slider :slides="bannersCollection" v-if="!!bannersCollection && bannersCollection.length > 0" />

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
      <div class="heading">
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Trending') }}</h2>
        </header>
      </div>
      <div class="row center-xs">
        <product-listing columns="4" :products="trendingCollection" />
      </div>
    </section>

    <section class="new-collection container mb-16" v-if="!!popularCollection && popularCollection.length > 0">
      <div class="heading">
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Popular Categories') }}</h2>
        </header>
      </div>
      <div class="row center-xs">
        <product-listing columns="4" :products="popularCollection" />
      </div>
    </section>

    <section class="new-collection container mb-16">
      <div class="heading">
        <header class="mb-6">
          <h2 class="text-h1 leading-h1 text-center">{{ $t('Installation Materials') }}</h2>
        </header>
      </div>
      <div class="row">
        <router-link
          :to="localizedRoute({ name: 'category', params: { id: installationMaterialsCategory.id, slug: installationMaterialsCategory.slug }})"
        >
          <img src="/assets/installation-materials.jpg" alt="">
        </router-link>
      </div>
    </section>

    <section class="new-collection container mb-16" style="margin-top: 100px;" v-if="!!installationMaterialsCollection && installationMaterialsCollection.length > 0">
      <div class="row center-xs">
        <product-listing columns="4" :products="installationMaterialsCollection" />
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

import { thumbnail } from '../../../../core/mixins/thumbnail'

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
    },
    bannersCollection () {
      return this.$store.state.homepage.banners_collection
    },
    installationMaterialsCollection () {
      return this.$store.state.homepage.installation_materials_collection
    },
    installationMaterialsCategory () {
      return this.$store.state.homepage.installation_materials_category
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
      let bannersQuery = prepareQuery({ queryConfig: 'banners' })
      let installationMaterialsQuery = prepareQuery({ queryConfig: 'installation-materials' })

      store.dispatch('category/list', { includeFields: config.entities.optimize ? config.entities.category.includeFields : null }).then((categories) => {
        store.state.homepage.installation_materials_category = categories.items.find(
          (category) => {
            return category.name === 'Installation Materials'
          }
        )
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
            store.dispatch('product/list', {
              query: installationMaterialsQuery,
              size: 8,
              sort: 'created_at:desc',
              includeFields: config.entities.optimize ? (config.products.setFirstVarianAsDefaultInURL ? config.entities.productListWithChildren.includeFields : config.entities.productList.includeFields) : []
            }).catch(err => {
              reject(err)
            }).then((res) => {
              if (res) {
                store.state.homepage.installation_materials_collection = res.items
              }
              store.dispatch('product/list', {
                query: bannersQuery,
                size: 8,
                sort: 'created_at:desc',
                includeFields: config.entities.optimize ? (config.products.setFirstVarianAsDefaultInURL ? config.entities.productListWithChildren.includeFields : config.entities.productList.includeFields) : []
              }).catch(err => {
                reject(err)
              }).then((res) => {
                if (res) {
                  store.state.homepage.banners_collection = res.items.map(
                    (item) => {
                      return {
                        image: thumbnail.methods.getThumbnail(item.image, 960, 320),
                        title: item.name,
                        link: item.banner_url,
                        button_text: 'Click Here'
                      }
                    }
                  )
                }
                return resolve()
              })
            })
          })
        })
      }).catch(err => {
        reject(err)
      })
    })
  }
}
</script>

<style>
.heading {
  background-color: #4fce76;
  border-radius: 4px;
}
.heading header {
  color: white;
}
.slide-content {
  height: 640px;
}
@media (max-width: 75em) {
  .main-slider {
    height: 400px;
  }
  .slide-content {
    height: 400px;
  }
}
@media (max-width: 64em) {
  .main-slider {
    height: 359px;
  }
  .slide {
    background-position: left;
  }
  .slide-content {
    height: 359px;
  }
}
</style>
