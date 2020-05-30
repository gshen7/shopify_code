# Add functionality to the cart page
This repository contains instructions for adding two features to Shopify cart pages. These instructions will require changes to be made to the code for the theme, so I would recommend duplicating the theme before as a backup. Once the changes are made to the code though, it can be configured from the theme settings afterward. Also note that these changes are for the Debut theme, and have not been tested with other themes. 

## Adding tipping on the cart page
![UI for tipping](/tipping/tip.png)

In the `tipping` folder, you'll find [instructions](/tipping/tipping_instructions.md) to add buttons that allow for a tip. While Shopify recently launched tipping in their checkout page, this adds the buttons to the bottom of the cart page. It works by adding a $1 tip product that you will have to create multiple times. However, it will not show as a product in your cart, instead showing after a subtotal and before the total cart amount. It also won't add to the cart count displayed on your cart icon. Finally, this will allow for you to set the percentage options for the tip buttons. 

## Adding a zip code checker on the cart page
![UI for shipping](/shipping/ship.png)

In the `shipping` folder, you'll find [instructions](/shipping/shipping_instructions.md) to add buttons that allow for a check to occur before the checkout button is presented. Note that this can be easily bypassed by navigating directly to the checkout url, or by modifying the page via a browser's developer tools. Also Shopify recently launched in my opinion, better local shipping restriction options in their checkout pages. However, I am making this available regardless, as it provides the shipping checker in the cart page instead, and also could be easily adapted to check for some other user input.

---

Feel free to create an issue if you'd like support with adding these to your Shopify theme.

---

If you found this helpful and want to support me, consider buying me a coffee!

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/U7U81Q15P)
