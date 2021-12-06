# Introduction

The goal of this demonstration is to provide an outline on how to filter products on a page, depending on which type of customer is logged in. Our example we will review the three main components needed for this functionality: Our GraphQL API Resource to retrieve product data, JavaScript to format and render the data, and Handlebars to identify the active customer group.

During the development of this solution, we wished to address the two concerns of scalability and performance. In order to accommodate for an increase in catalog size, we aimed to keep the amount of network requests to a minimum to ensure site speed remains high. As we will see later, this is accomplished by specifying the product IDs we wish to retrieve within our GraphQL query.

</br>

The source code for the demo can be found at: https://www.djawidjawijdaw.com  
A live example can be viewed at: https://spseprojectstore.mybigcommerce.com

</br>

To test this, you may use the following customer account credentials:
```
Username: customer1
Password: asdfasdf1234

Username: customer2
Password: asdfasdf1234

Username: customer3
Password: asdfasdf1234
```
# Retrieve Customer Group and Formulate Query

To begin, we will declare variables to store data needed for our query:

```
let product_ids = null;
let customer_group = {{customer.customer_group_id}};
```
</br>

From here, we can create a switch expression to assign a new value to our product_ids variable, depending on the active customer group.

```
switch(customer_group) {
  case 1:
    product_ids = [112];
    break;
  case 2:
    product_ids = [113];
    break;
  case 3:
    product_ids = [114];
    break;
 }
```

</br>

Using the product_ids array, we can then pass this as an argument into our GraphQL query to ensure that only matching products are requested. This can be seen on the third line of the code sample below.

              
```
const graphQLQuery = queryProductsbyCustomerGroup {
  site {
    products${params.product_ids ? `(entityIds:[${product_ids}])` : ''} {
      edges {
        product: node {
          ...ProductFields
        }
      }
    }
  }
}
```
# Sending the Query and Rendering Products

The GraphQL query can be sent to our Storefront API by sending a request through the Fetch API and utilizing our {{settings.storefront_api.token}} handlebars object for authentication.

```
fetch(graphQLUrl, {
              method: 'POST',
              credentials: 'include',
              mode: 'cors',
              headers: { 'Content-Type': 'application/json',
                       'Authorization': `Bearer {{settings.storefront_api.token}}`},
              body: JSON.stringify({ query: graphQLQuery}),
}) 
```
</br>

From there, the individual product attributes may be obtained and rendered onto DOM elements using your frontend language of choice. For our example, we had used JavaScript to declare a function that populates DOM elements into product cards.

```
function renderProduct(product, addToCartURLFormat) {
            return `
                <div class="card" style="min-width: 25%;">
                ${product.defaultImage ? `<a href= "${product.path}"> <img loading="lazy" class="card-img-top" style="min-height: 25%; object-fit: contain;" src="${product.defaultImage.img960px}" srcset="${renderSrcset(product.defaultImage)}" alt="${product.defaultImage.altText}"></a>` : ''
                 }
                <div class="card-body">
                  <h5 class="card-title">${product.name} ${renderPrice(product.prices)}</h5>
                  <p class="card-text text-truncate">${stripHtml(product.description)}</p>
                  
                  <a href="${addToCartURLFormat}${product.entityId}" class="btn btn-primary">Add to cart</a>
                </div>
              </div>`
        }
```

# Conclusion

Through the use GraphQL, we can ensure that site performance is maintained by specifying which products are included in our query. Our example uses simple arrays to represent the relationship between products and customer groups, however this model can accommodate for external product lists such as those taken from an ERM or PIP. Additionally, by minimizing the use of Handlebars objects, we empower developers to utilize the front end language of their choice.
