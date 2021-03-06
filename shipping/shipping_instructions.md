### Adding configurable settings in `settings_schema.json`

Add to bottom:
```
{
    "name": "Shipping Checker",
    "settings": [
      {
        "type": "select",
        "id": "shipping_checker",
        "label": "Show the shipping checker?",
        "options": [
          {
            "value": "Disabled",
            "label": "No"
          },
          {
            "value": "Enabled",
            "label": "Yes"
          }
        ],
        "default": "Enabled"
      },
      {
        "type": "select",
        "id": "shipping_checker_heading_show",
        "label": "Show the heading text?",
        "options": [
          {
            "value": "Disabled",
            "label": "No"
          },
          {
            "value": "Enabled",
            "label": "Yes"
          }
        ],
        "default": "Enabled"
      },
      {
        "type": "text",
        "id": "shipping_checker_heading",
        "label": "Heading text",
        "default": "Check if we take orders from your address"
      },
      {
        "type": "text",
        "id": "shipping_checker_submit_button_label",
        "label": "Submit button label",
        "default": "Check"
      },
      {
        "type": "text",
        "id": "shipping_checker_submit_button_label_pending",
        "label": "Submit button label when checking",
        "default": "Checking..."
      },
      {
        "type": "text",
        "id": "shipping_checker_fail_message",
        "label": "Message if unable to ship",
        "default": "Unfortunately we're unable to ship to this address"
      },
      {
        "type": "text",
        "id": "shipping_checker_success_message",
        "label": "Message if able to ship",
        "default": "We are able to ship to this address. Proceed to checkout."
      },
      {
        "type": "textarea",
        "id": "shipping_checker_codes",
        "label": "Allow shipping to these postal\/zip codes (* represents any letter, # represents any number). Enter as comma separated values.",
        "default": "E1X #*#"
      },
      {
        "type": "text",
        "id": "shipping_checker_inactive_color",
        "label": "Hex code color code for base button",
        "default": "bbbbbb"
      },
      {
        "type": "text",
        "id": "shipping_checker_active_color",
        "label": "Hex code color code for button when it is moused over",
        "default": "ffffff"
      }
    ]
  },
```

### Add shipping-checker snippet
```
<br/>
{% unless settings.shipping_checker_header_show == 'Disabled' %}
<h5>{{ settings.shipping_checker_heading | default: 'Check if we take orders from your address' }}</h5>

{% endunless %}
{% unless settings.shipping_checker == 'Disabled' %}

<div id="shipping-checker">
  <div class="container">
    <span class="ship-check">
      <label class="check-ship__label check-ship">{{ settings.shipping_checker_submit_button_label | default: 'Check' }}</label>
      <input class="check-ship__input " placeholder="Postal Code" type="text" id="address_zip" name="address[zip]" {% if shop.customer_accounts_enabled and customer %} value="{{ customer.default_address.zip }}"{% endif %}/>
    </span>
  </div>
  <br>
  <div class="wrapper hidden" id="checkout-success" >{{settings.shipping_checker_success_message}}</div>
  <div class="wrapper hidden" id="checkout-fail" >{{settings.shipping_checker_fail_message}}</div>
</div>

{% endunless %}

<div class="cart__buttons-container">
  <div class="cart__submit-controls">
    {%- unless section.settings.cart_ajax_enable -%}                
    <a name="checkout" class="cart__submit btn btn--secondary" href="/cart/update">{{'cart.general.update' | t}}</a> 
    {%- endunless -%}
    <a name="checkout" class="cart__submit btn btn--small-wide {% unless settings.shipping_checker == 'Disabled' %}hidden{%- endunless -%}" href="/cart/checkout">{{'cart.general.checkout' | t}}</a> 


  </div>

  <div class="cart__error-message-wrapper hide" role="alert" data-cart-error-message-wrapper>
    <span class="visually-hidden">{{ 'general.accessibility.error' | t }} </span>
    {% include 'icon-error' %}
    <span class="cart__error-message" data-cart-error-message></span>
  </div>

  {%- if additional_checkout_buttons -%}
  <div class="additional-checkout-buttons" >{{ content_for_additional_checkout_buttons }}</div>
  {%- endif -%}
</div>
<br/>
<p> Deliveries are made every Friday for orders placed before 4:00pm on Thursday. For orders containing Raani Foods products, orders placed after 9:30am on Wednesday will be delivered the following week as these products are made fresh. </p>


<!--[if lte IE 8]>
<style> #shipping-checker { display: none; } </style>
<![endif]-->

<script>
  
  jQuery(function($) {
    $(document).ready(function() {
      $("#dynamic-checkout-cart").addClass("hidden");
    });
    _enableCheckout = function() {
      $("#checkout-fail").addClass("hidden");
      $("#checkout-success").removeClass("hidden");
      $("[name='checkout']").removeClass("hidden");
      $("#dynamic-checkout-cart").removeClass("hidden");
    },
      _failCheckout = function() {
      $("[name='checkout']").addClass("hidden");
      $("#checkout-success").addClass("hidden");
      $("#checkout-fail").removeClass("hidden");
      $("#dynamic-checkout-cart").addClass("hidden");
    },
      _hideMessages = function(){
      $("[name='checkout']").addClass("hidden");
      $("#checkout-success").addClass("hidden");
      $("#checkout-fail").addClass("hidden");
      $("#dynamic-checkout-cart").addClass("hidden");
      $(".check-ship").text("{{ settings.shipping_checker_submit_button_label_pending | default: 'Check' }}");
    },
      _enableButton = function(){
      $(".check-ship").text("{{ settings.shipping_checker_submit_button_label | default: 'Checking...' }}");
    },
      _checkCartShippingValidity = function(zip, codesStr) {
      var codes = codesStr.split(",")

      zip = zip.replace(/\s/g, '')
      zip = zip.toUpperCase()

      for (var i=0; i<codes.length; i++){
        code = codes[i]
        code = code.replace(/\s/g, '')
        code = code.toUpperCase();

        code = code.replace(/\*/g, "[a-zA-Z]")
        code = code.replace(/#/g, "[0-9]")

        var codeRegex = new RegExp(code)

        if(zip.match(codeRegex)){
          _enableCheckout()
          return
        }
      }

      _failCheckout()
    },
    _updateCheckout = function(){
      var shippingcodes = {{settings.shipping_checker_codes | default: "" | json}};
      _hideMessages()
      setTimeout(function(){
        var a = {};
        a.zip = $("#address_zip").val() || "";
        _checkCartShippingValidity(a.zip,shippingcodes)
        _enableButton()
      },250)
    };
  });
  var checkShip = $(".check-ship");
  checkShip.bind("click", function() {
    _updateCheckout()
  });
  var checkShipInput = $(".check-ship__input");
  // If Enter key is pressed while input box has focus, update donation.
  checkShipInput.bind('keypress', function(evt) {
    if (evt.which === 13) {
      _updateCheckout();
      return false;
    }
  });
  // If Donation amount has changed, update donation.
  checkShipInput.bind('change, blur', function() {
    _updateCheckout();
  });
</script>

<style>
  #shipping-checker > h4{
    margin-top: 25px;
  }
  .check-ship__input {
    box-sizing: border-box;
    border: 0 !important;
    display: inline-block;
    padding: 8px 12px !important;
    letter-spacing: 1px;
    font-weight: 400;
    font-size: 15px !important;
    line-height: 22px !important;
  }
  .check-ship {
    box-sizing: border-box;
    float: right;
    margin: 0 0px 0px 0;
    position: relative;
    border: 1px solid #{{settings.shipping_checker_active_color}};
    background-color: #{{settings.shipping_checker_inactive_color}};
    transition: background-color .5s;
    color: #000000;  
    cursor: pointer;
    display: inline-block;
    padding: 8px 12px !important;
    letter-spacing: 1px;
    font-weight: 400;
    font-size: 15px !important;
    line-height: 24px !important;
    top: -1px;
    right: 1px;
    width: 110px;
    text-align: center;
  }
  .check-ship .check-ship__input {
    float: right;
    margin-left: -1px;
  }
  .check-ship__label:hover {
    background-color: #{{settings.shipping_checker_active_color}};
    border: 1px solid #{{settings.shipping_checker_inactive_color}};
    color: #000000;
  }
  #address_zip{
    height: 40px !important;
  }
</style>
```

### Inserting shipping checker snippet and stopping enter from proceeding to checkout in `cart-template.liquid`
Find the `form` open and close tags. Replace them with `<div novalidate class="cart">` and `</div>

Find the div with class `cart__buttons-container`. Replace it with `{% render 'shipping-checker' %}`.