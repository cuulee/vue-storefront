# Introduction

Below you can find solutions for most common problems and advises for typical config changes required by Vue Storefront.
If you solved any new issues by yourself please let us know on [slack](http://vuestorefront.slack.com) and we will add them to the list so others don't need to reinvent the wheel.

# Question

### <a name="products-not-displayed"></a>Product not displayed (illegal_argument_exception)

See discussion in [#137](https://github.com/DivanteLtd/vue-storefront/issues/137)

### <a name="dynamic-pricing"></a>How to get dynamic prices to work (catalog rules)

After following the Tutorial on [how to connect to Magento2](https://medium.com/@piotrkarwatka/vue-storefront-how-to-install-and-integrate-with-magento2-227767dd65b2) the pricess are updated just after manually runing [mage2vuestorefront cli command](https://github.com/DivanteLtd/mage2vuestorefront).

However there is an option to get the prices dynamicaly. To do so you must change the config inside `conf/local.json` from the default (`conf/default.json`):

```json
"products": {
      "preventConfigurableChildrenDirectAccess": true,
      "alwaysSyncPlatformPricesOver": false,
      "clearPricesBeforePlatformSync": false,
      "waitForPlatformSync": false,
      "endpoint": "http://localhost:8080/api/product"
    },
    ```
    
to:

```json
"products": {
      "preventConfigurableChildrenDirectAccess": true,
      "alwaysSyncPlatformPricesOver": true,
      "clearPricesBeforePlatformSync": true,
      "waitForPlatformSync": false,
      "endpoint": "http://localhost:8080/api/product"
    },
```

To make it work you need have Magento2 oauth keys konfigured in your `vue-storefront-api` - `conf/local.json`.
This change means that each time product list will be displayed, VS will get the fresh prices directly from magento without the need to re-index ElasticSearch.

### <a name="no-products"></a>No products found! after node --harmony cli.js fullreindex

Take a look at the discussion at [#644](https://github.com/DivanteLtd/vue-storefront/issues/644)
Long story short -> you need to run the following command within the `mage2nosql` project:

```bash
node cli.js products --partitions=1
```

### <a name="sync-carts"></a>How to sync the products cart with Magento to get the Cart Promo Rules up and runnig

To display the proper prices and totals after Magento calculates all the discounts and taxes you need to modify the `conf/local.json` config (for a reference take a look at `conf/default.json`) by putting there a additional section:

```json
  "cart": {
    "synchronize": true,
    "synchronize_totals": true,
    "create_endpoint": "http://localhost:8080/api/cart/create?token={{token}}",
    "updateitem_endpoint": "http://localhost:8080/api/cart/update?token={{token}}&cartId={{cartId}}",
    "deleteitem_endpoint": "http://localhost:8080/api/cart/delete?token={{token}}&cartId={{cartId}}",
    "pull_endpoint": "http://localhost:8080/api/cart/pull?token={{token}}&cartId={{cartId}}",
    "totals_endpoint": "http://localhost:8080/api/cart/totals?token={{token}}&cartId={{cartId}}"
  },  
```

To make it work you need have Magento2 oauth keys konfigured in your `vue-storefront-api` - `conf/local.json`.
After this change you need to restart the `npm run dev` or `npm run` command to take the config changes into consideration by the VS. All the cart actions (add to cart, remove from cart, modify the qty) are now synchronized directly with Magento2 - for both: guest and logged in clients.
