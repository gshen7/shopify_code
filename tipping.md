## Tip functionality

### Make a tip product (dont track quantity, make it available in store - you'll have to make a collection that consists of all products without the tip product for it to not show when browsing products, make it 1 dollar, make it have Tip as vendor

### Create a navigation menu called tip, containing only the Tip product

### Adding configurable settings in `settings_schema.json`

Add to bottom:
```
{
    "name": "Tipping",
    "settings": [
        {
        "type": "select",
        "id": "tipping_option",
        "label": "Tip option display",
        "options": [
            {
            "value": "Disabled",
            "label": "No Tip Options"
            },
            {
            "value": "Percent",
            "label": "Percents and Custom"
            },
            {
            "value": "Amount",
            "label": "Amounts and Custom"
            }
            
        ],
        "default": "Percent"
        },
        {
        "type": "text",
        "id": "tipping_heading",
        "label": "Heading text",
        "default": "Support us with a tip!"
        },
        {
        "type": "textarea",
        "id": "tipping_bases",
        "label": "Tipping base percents\/amounts (descending order)",
        "default": "5,10,15"
        },
        {
        "type": "text",
        "id": "tipping_vendor",
        "label": "Tipping vendor name",
        "default": "Tip"
        },
        {
        "type": "text",
        "id": "tipping_menu",
        "label": "Tipping menu handle",
        "default": "tip"
        },
        {
          "type": "text",
          "id": "tipping_inactive_color",
          "label": "Hex code color code for base button",
          "default": "bbbbbb"
        },
        {
          "type": "text",
          "id": "tipping_active_color",
          "label": "Hex code color code for button when it is moused over",
          "default": "97bd62"
        },
        {
          "type": "text",
          "id": "tipping_hover_color",
          "label": "Hex code color code for button when it is moused over",
          "default": "ffffff"
        }
    ]
}
  ```

### Making the cart count bubble not show count of tip product added in `header.liquid`

Find the div with id `CartCount`, and replace inner html:
```
{% assign cart_count = 0 %}
{% for item in cart.items %}
{% unless item.vendor == settings.tipping_vendor %}
{% assign cart_count = cart_count | plus: item.quantity %}
{% endunless %}
{% endfor %}
<span data-cart-count>{{ cart_count }}</span>
<span class="icon__fallback-text medium-up--hide">{{ 'layout.cart.items_count' | t: count: cart_count }}</span>
```

### Add tip snippet
```
{% unless settings.tipping_options == "Disabled" %}
<!-- checking if cart has a tip currently -->
{% assign tip_amount = 0 %}
{% assign tip_index = -1 %}
{% for item in cart.items %}
{% if item.vendor == settings.tipping_vendor %}
{% assign tip_amount = item.quantity | times:100%}
{% assign tip_index = forloop.index0 %}
{% endif %}
{% endfor %}

{% assign variantId = linklists[settings.tipping_menu].links.first.object.first_available_variant.id %}

<!-- calculate what amounts correspond to what percents -->
{% assign bases = settings.tipping_bases | replace:' ','' | split: ',' %}
{% assign amounts = '' | split: '' %}
{% assign cent_amounts = '' | split: '' %}

{% if settings.tipping_option == "Percent" %}

	{% assign set_percent_in_cart = -1 %}
	{% for base in bases %}

		{% assign amount = tip_amount | times:-1 | plus:cart.items_subtotal_price | times: 0.01 | times: base | divided_by:100 | floor | split: "," %}
		{% assign amounts = amounts | concat: amount %}

		{% assign cent_amount = amount | join: '' | plus:0 | times: 100 %}
		{% if tip_amount == cent_amount %}
			{% assign set_percent_in_cart = base | plus:0 %}
		{% endif %}

	{% endfor %}

{%elsif settings.tipping_option == "Amount"%}
	{% assign set_amount_in_cart = -1 %}
	{% for base in bases %}

		{% assign cent_amount = base | times: 100 %}
		{% if tip_amount == cent_amount %}
			{% assign set_amount_in_cart = base | plus:0 %}
		{% endif %}

	{% endfor %}

{% endif %}

<div class="cart-subtotal tip-fee {% if tip_index < 0 %}hidden{%endif%}">
  <span class="cart-subtotal__title">{{ 'cart.general.subtotal' | t }}</span>            
  <span class="cart-subtotal__price" data-cart-subtotal-less-tip>{{ cart.total_price | minus:tip_amount | money_with_currency }}</span>
</div>

<div class="cart-subtotal tip-fee {% if tip_index < 0 %}hidden{%endif%}">
  <span class="cart-subtotal__title">Tip (Thanks for your support!)</span>            
  <span class="cart-subtotal__price" data-cart-tip-display>{{ tip_amount | money_with_currency }}</span>
</div>

<hr class="total tip-fee {% if tip_index < 0 %}hidden{%endif%}">

<div class="cart-subtotal">
  <span class="cart-subtotal__title">Total</span>            
  <span class="cart-subtotal__price" data-cart-subtotal>{{ cart.total_price | money_with_currency }}</span>
</div>

{%- capture taxes_shipping_checkout -%}
{%- if shop.taxes_included and shop.shipping_policy.body != blank -%}
{{ 'cart.general.taxes_included_and_shipping_policy_html' | t: link: shop.shipping_policy.url }}
{%- elsif shop.taxes_included -%}
{{ 'cart.general.taxes_included_but_shipping_at_checkout' | t }}
{%- elsif shop.shipping_policy.body != blank -%}
{{ 'cart.general.taxes_and_shipping_policy_at_checkout_html' | t: link: shop.shipping_policy.url }}
{%- else -%}
{{ 'cart.general.taxes_and_shipping_at_checkout' | t }}
{%- endif -%}
{%- endcapture -%}
<div class="cart__shipping rte">{{ taxes_shipping_checkout }}</div>

<div class="grid__item text-right"> 

<div class="clearfix">

  <h5>{{ settings.tipping_heading }}</h5>
  
  <input class="hidden{% if tip_index == -1 %} active{% endif %} remove-tip" type="radio" name="remove-tip"  value="0" />
  <label data-cart-tip class="remove-tip__label tip remove-tip" id="remove-tip" amount="0" for="remove-tip">No Tip</label>

  {% if settings.tipping_option == "Percent" %}
  	{%if set_percent_in_cart > 0 %}
  		{% assign base_selected = true %}
  	{%else%}
  		{% assign base_selected = false %}
  	{%endif%}
  {%elsif settings.tipping_option == "Amount"%}
  	{%if set_amount_in_cart > 0 %}
  		{% assign base_selected = true %}
	{%else%}
  		{% assign base_selected = false %}
  	{%endif%}
  {% endif %}
  
  <span class="tip-variable-amount">
    <input class="hidden{% unless base_selected or tip_index == -1 %} active{% endunless %}"  type="radio" name="tip-variable-amount"  value="" />
    <label data-cart-tip class="tip-variable-amount__label disabled tip" id="tip-variable" for="tip-variable">Custom</label>
    <input class="tip-variable-amount__input " placeholder="Custom Tip"{% unless tip_index == -1 %} value="{{ tip_amount | money_without_trailing_zeros }}"{% endunless %} type="text" onkeypress="return event.charCode >= 48 && event.charCode <= 57"/>
  </span>

  {% if settings.tipping_option == "Percent" %}
    {% for base in bases %}
      {%assign amount_for = amounts[forloop.index0] | plus:0 %}
      {% assign base_int = base | plus:0 %}
      <input class="hidden{% if set_percent_in_cart == base_int and tip_index != -1 %} active{% endif %} tip-fixed" type="radio" name="tip-fixed-{{base}}"/>
      <label data-cart-tip class="tip {% if amount_for <= 0 %}hidden{% endif %}" id="tip-fixed-{{base}}" amount="{{amounts[forloop.index0]}}" for="tip-fixed-{{base}}">{{base}}% ({{ amounts[forloop.index0] | times:100 | money_without_trailing_zeros }})</label>
    {% endfor %}
  {% elsif settings.tipping_option == "Amount" %}
  	{% for base in bases %}
      {% assign base_int = base | plus:0 %}
      <input class="hidden{% if set_amount_in_cart == base_int and tip_index != -1 %} active{% endif %} tip-fixed" type="radio" name="tip-fixed-{{base}}"/>
      <label data-cart-tip class="tip" id="tip-fixed-{{base}}" amount="{{base}}" for="tip-fixed-{{base}}">${{base}}</label>
    {% endfor %}
  {% endif %}


</div>
</div>

<script>
  theme.strings = {
    tipping_variant: {{ variantId | default: '' | json }}
  }
</script>

<style> 
  .hidden {
    display: none !important;
  }
  .tip {
    box-sizing: border-box;
    position: relative;
    float: right;
    margin: 0 10px 10px 0;
    background-color: #{{settings.tipping_inactive_color}};
    border: 1px solid #{{settings.tipping_inactive_color}};
    transition: background-color .5s;
    color: '#ffffff';  
    cursor: pointer;
  }
  .remove-tip {
    margin: 0 0px 0px 0 !important;
  }
  .tip, .tip-variable-amount__input {
    box-sizing: border-box;
    display: inline-block;
    padding: 8px 12px !important;
    letter-spacing: 1px;
    font-weight: 400;
    font-size: 15px !important;
    line-height: 22px !important;
  }
  .tip-variable-amount {
    box-sizing: border-box;
    float: right;
    margin: 0 10px 10px 0;
    padding: 0;
  }
  .tip-variable-amount .tip-variable-amount__input {
    float: right;
    margin-left: -1px;
  }
  .tip-variable-amount__label { 
    box-sizing: border-box; 
    margin: 0;
  }
  .tip-variable-amount__input {
    box-sizing: border-box;
    width: 120px !important;
    height: 40px !important;
    margin: 0;
    border: 1px solid #{{settings.tipping_inactive_color}};
    border-radius: 1px !important;
  }
  .tip-variable-amount__input--disabled {  
    opacity: 0.5;
  }
  input:checked ~ .tip-variable-amount__input, 
  input.active ~ .tip-variable-amount__input {
    border: 1px solid #{{ settings.tipping_active_color }} !important;
  }
  input:checked + .tip, input.active + .tip {
    background-color: #{{settings.tipping_active_color}};
    border: 1px solid #{{settings.tipping_active_color}};
  }
  .tip:hover {
    border: 1px solid #{{settings.tipping_inactive_color}};
    background-color: #{{settings.tipping_hover_color}};
  }
  input:checked + .tip:after,
  input.active + .tip:after {
    position: absolute;
    font-size: 12px;
    top: -2px;
    right: 4px;
  }

  form.cart > table {
    margin-bottom: 10px !important;
  }
  hr.total {
    margin: 10px 0px !important;
  }

</style>
{%endunless%}
```

### Insertting tip snippet into `cart-template.liquid`

Find the div with class `cart-subtotal`. Near it, will also be a div with class `cart_shipping rte`. Delete both of these divs. Put `{% render 'tip' %}` where the `cart-subtotal` div used to be.

### Adding script for controlling the tip from the tip buttons

Rename the theme.js file to theme.js.liquid

Find `selectors` object defined within `theme.Cart`. At the bottom, add:
```
tipInput: '.tip-variable-amount__input',
cartSubtotalLessTip: '[data-cart-subtotal-less-tip]', 
cartTip: '[data-cart-tip]', 
cartTipDisplay: '[data-cart-tip-display]', 
cartButtonsContainer: '.cart__buttons-container',
tipFee: '.tip-fee'
```

Find `this._setupCartTemplates();`. Above it, add:
```
this.$container.on(
    'click',
    selectors.cartTip,
    this._handleInputTip.bind(this)
);
this.$container.on(
    'keypress',
    selectors.tipInput,
    this._handleKeypressInputTip.bind(this)
);
this.$container.on(
    'change,blur',
    selectors.tipInput,
    this._handleInputTip.bind(this)
);
```

Find `_setupCartTemplates: function() {`. At the top of the function, add:
```
var templates = $(selectors.cartItem, this.$container)
for(var i = 0; i<templates.length;i++){
    if(templates[i].getAttribute('data-cart-item-url')=="/products/tip-jar-app?variant=32258138341472"){
        continue;
    } else {
        this.$itemTemplate = $(templates[i]).clone()
        break;
    }                         
}
```
Underneath the function, add:
```
//rerender tip buttons
_rebuildTip: function(state, cart_count){
    var bases = "{{settings.tipping_bases}}".replace(/\s/g,'').split(",")
    var amount_for={}
    var found = false
    for(var i=0;i<bases.length;i++){
    if("{{settings.tipping_option}}"=="Percent"){
        amount_for[bases[i]]=Math.floor((state.total_price-cart_count*100)*0.01*bases[i]/100)
        $('#tip-fixed-'+bases[i]).html(bases[i]+"% ($"+amount_for[bases[i]]+")")    
    } else if("{{settings.tipping_option}}"=="Amount"){
        amount_for[bases[i]]=bases[i]
        $('#tip-fixed-'+bases[i]).html("$"+amount_for[bases[i]])    
    }

    $('#tip-fixed-'+bases[i]).attr("amount",amount_for[bases[i]])    

    if(amount_for[bases[i]]<=0){
        $('#tip-fixed-'+bases[i]).addClass("hidden")  
    }else {
        $('#tip-fixed-'+bases[i]).removeClass("hidden")  
    }

    if(!found){

        if(amount_for[bases[i]] != 0 && amount_for[bases[i]]==cart_count){
        $('.tip-fixed').removeClass("active")  
        $('[name="tip-fixed-'+bases[i]+'"]').addClass("active")  
        $('[name="tip-variable-amount"]').removeClass("active") 
        $('[name="remove-tip"]').removeClass("active")   
        found=true;
        }else if(cart_count>0){
        $('.tip-fixed').removeClass("active")  
        $('[name="tip-variable-amount"]').addClass("active") 
        $('[name="remove-tip"]').removeClass("active")    
        } else {
        $('.tip-fixed').removeClass("active")  
        $('[name="tip-variable-amount"]').removeClass("active") 
        $('[name="remove-tip"]').addClass("active")    
        }
    }
    }
},
//handle tip button inputs
_handleKeypressInputTip: function(evt){
    if (evt.which === 13) {
    this._handleInputTip(evt);
    return false;
    }
},
_handleInputTip: function(evt) {
    var $input = $(evt.target);
    var value = parseInt($input.attr("amount"));
    var updateData = { updates: {} }
    var variantId = theme.strings.tipping_variant 

    if(value>=0){
    qty = value
    }else {
    qty = parseInt($(selectors.tipInput).val().replace(/[^0-9.,]/,''), 10) || 0;
    }

    updateData['updates'][variantId+""] =  qty;
    
    $(selectors.tipInput).val("$"+qty)
    
    var params = {

    url: '/cart/update.js',
    data: updateData,
    dataType: 'json'
    };

    $(selectors.cartButtonsContainer).addClass("hidden")
    
    $.post(params)
    .done(
    function(state) {
        var cart_count = state.items.filter(item => item.vendor == "{{settings.tipping_vendor}}").reduce((a,b) => a + b.quantity,0) || 0 

        if (state.item_count === 0) {
        this._emptyCart();
        } else {
        this._createCart(state);
        this._rebuildTip(state, cart_count);
        }

        this._setCartCountBubble(state.item_count-cart_count);
        $(selectors.cartButtonsContainer).removeClass("hidden")
    }.bind(this)
    )
    .fail(
    function() {
        //failure
    }.bind(this)
    );
},
```

Find `_updateItemQuantity: function(`. Change the `.done` callback function after the `post` request is called to:
```
.done(
    function(state) {
    var cart_count = state.items.filter(item => item.vendor == "{{settings.tipping_vendor}}").reduce((a,b) => a + b.quantity,0) || 0 //figure out how much tip is in cart
    if (state.item_count === 0 || state.item_count == cart_count) {
        window.location.href = '/cart/clear' //clear the cart if only tips left in cart
    } else {
        this._createCart(state);

        if (value != 0) {
        var $lineItem = $('[data-cart-item-key="' + key + '"]');
        var item = this.getItem(key, state);

        $(selectors.quantityInput, $lineItem).focus();
        this._updateLiveRegion(item);
        }
        
        this._rebuildTip(state, cart_count); //rebuild tips 
    }

    this._setCartCountBubble(state.item_count-cart_count); //set the cart count bubble to not include tips
    }.bind(this)
)
```

Find `_createCart: function(state) {`. At the bottom of the function, add:
```
//render the subtotal and tip as a fee lines if needed
var cart_count = state.items.filter(item => item.vendor == "{{settings.tipping_vendor}}").reduce((a,b) => a + b.quantity,0) || 0

$(selectors.cartSubtotalLessTip, this.$container).html(
theme.Currency.formatMoney(
    state.total_price - cart_count*100,
    theme.moneyFormatWithCurrency
)
);

$(selectors.cartTipDisplay, this.$container).html(
theme.Currency.formatMoney(
    cart_count*100,
    theme.moneyFormatWithCurrency
)
);

if(cart_count>0){
$(selectors.tipFee, this.$container).removeClass("hidden")

} else {
$(selectors.tipFee, this.$container).addClass("hidden")
}
```

Find `_createLineItemList: function(state) {`. At the top of the callback function within the map function, add:
```
//dont render a line item for the tip product
var variantId = theme.strings.tipping_variant 
if(item.vendor=="{{settings.tipping_vendor}}"){
    let $item = $("selector[data-cart-item-url*='"+variantId+"']",this.$container).clone();
    return $item[0];
}
```

Find `_setItemRemove: function($item, title) {`. Replace it with:
```
_setItemRemove: function($item, title) {
    // added if to account for no remove for tip item
    if(theme.strings.removeLabel){
        $(selectors.cartRemove, $item).attr(
            'aria-label',
            theme.strings.removeLabel.replace('[product]', title)
        );
    }
},
```

Find `_onRemoveItem_: function(evt) {`. Change the `.done` callback function after the `post` request is called to:
```
.done(
    function(state) {
    var cart_count = state.items.filter(item => item.vendor == "{{settings.tipping_vendor}}").reduce((a,b) => a + b.quantity,0) || 0
    if (state.item_count === 0 || state.item_count == cart_count) {
        window.location.href = '/cart/clear' 
    } else {
        this._createCart(state);
        
        this._rebuildTip(state, cart_count);
    }

    this._setCartCountBubble(state.item_count-cart_count);
    }.bind(this)
)
```

Find `_formatRemoveMessage: function(lineItem) {`. Replace it with:
```
_formatRemoveMessage: function(lineItem) {
    var quantity = lineItem.data('cart-item-quantity');
    var url = lineItem.attr(attributes.cartItemUrl);
    var title = lineItem.attr(attributes.cartItemTitle);

    //added if to account for no remove for tip item
    if(theme.strings.removedItemMessage){
        return theme.strings.removedItemMessage
        .replace('[quantity]', quantity)
        .replace(
            '[link]',
            '<a ' +
            'href="' +
            url +
            '" class="text-link text-link--accent">' +
            title +
            '</a>'
        );
    }
},
```

Find `_updateCartPopupContent: function(item) {`. Change the `.then` callback function after the `getJSON` request to:
```
.then(
    function(cart) {
        //for removing tip from number
        var cart_count = cart.items.filter(item => item.vendor == "{{settings.tipping_vendor}}").reduce((accumulator, current) => accumulator + current.quantity,0) || 0
        this._setCartQuantity(cart.item_count-cart_count);
        this._setCartCountBubble(cart.item_count-cart_count);
        this._showCartPopup();
    }.bind(this)
);
```