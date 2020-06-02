# Approach
The approach for discovering how to use the `shopifyAPI` python library was to map the endpoints from the [REST API reference](https://shopify.dev/docs/admin-api/rest/reference)

## Rate Limit
* I think the rate limit was around 2 requests per second, but I didn't run into issues with it often
* I found it faster to pull all the data I would need initially though; ie. rather than make a request whenever I needed a product object, to just get all product objects initially and find what I needed in memory

## Auth
* One of the few aspects of the API that is covered in the getting started guide, but I'll repeat it here anyways (note that the code below is only for a private app)
```
shop_url = "https://%s:%s@care-for-toronto.myshopify.com/admin" % ("APIKEY", "PASSWORD")
shopify.ShopifyResource.set_site(shop_url)
```

## shopify.Resource
* Used to access the base endpoints for resources available in the REST API (ex. `shopify.Order`, `shopify,InventoryItem`, `shopify.Product`)

## shopify.Resource.find()
* Used to perform the GET queries for that resource, with arguments allowed for each  (ex. `shopify.Order.find(ids="comma separated ids", created-at_min="date with format specified in REST reference"))
* Most of these will return as paginated collections. I found it useful to convert these to arrays of dictionaries. The code below request all pages of products and converts it to an array of dictionaries.
```
resourceObj = shopify.Product

data = []
page = 1
while True:
    resource = resourceObj.find(limit=250, page=page)
    data.extend(resource)
    if len(resource) != 250:
        break
    page += 1

resources = [r.to_dict() for r in data]
products = resources
```

## Other specifics
### finding inventory items
* I found it best to request these by inventory item id, obtained from the list of all products
* the `.find` function with `ids` parameter only takes a list of up to 100 ids though, so the code below splits it into evenly sized pages as close to 100 as possible
```
inventoryIds = []
for product in products:
    for variant in product["variants"]:
        inventoryIds.append(variant["inventory_item_id"])

inventoryIds = numpy.array_split(numpy.array(inventoryIds),int(len(inventoryIds)/100)+1)

resourceObj = shopify.InventoryItem

data = []
page = 1
for i in range(0,len(inventoryIds)):
    resource = resourceObj.find(limit=250, ids=','.join([str(num) for num in inventoryIds[i]]))
    data.extend(resource)

resources = [r.to_dict() for r in data]
inventory = resources
```
