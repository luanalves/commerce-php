---
title: Add Custom Validations Before Order Placement | Commerce PHP Extensions
description: Follow this tutorial to create programmatic validations that check orders before they are placed in the Adobe Commerce and Magento Open Source checkout experience.
---

# Add custom validations before order placement

This topic describes how to add custom validations to be performed before the order is placed during checkout. Namely, the validations which are performed after a shopper clicks the **Place Order** button. Writing the validation logic itself is not covered in this topic.

To add custom validations before the order placement action, you must do the following:

1. [Create the validator](#step-1-create-the-validator).
1. [Add validator to the validators pool](#step-2-add-validator-to-the-validators-pool).
1. [Declare the validation in the checkout layout](#step-3-declare-the-validation-in-the-checkout-layout).

## Step 1: Create the validator

For the sake of compatibility, upgradability and easy maintenance, do not edit the default application code, add your customizations in a separate module. For your checkout customization to be applied correctly, your custom module should [depend](../../../development/build/composer-integration.md) on the `Magento_Checkout` module. Do not use `Ui` for your custom module name, because `%Vendor%_Ui` notation, required when specifying paths, might cause issues.

In your custom module directory, create a `.js` file implementing the validator. It should be located under `<your_module_dir>/view/frontend/web/js/model` directory.

Following is a sample of the validator `.js` file. It must necessarily implement the `validate()` method:

```js
define(
    ['mage/translate', 'Magento_Ui/js/model/messageList'],
    function ($t, messageList) {
        'use strict';
        return {
            validate: function () {
                const isValid = false; //Put your validation logic here

                if (!isValid) {
                    messageList.addErrorMessage({ message: $t('a possible failure message ...  ') });
                }

                return isValid;
            }
        }
    }
);
```

## Step 2: Add validator to the validators pool

Your custom validator must be added to the pool of "additional validators". To do this, in the `<your_module_dir>/view/frontend/web/js/view` directory create a new `<your-validation>.js` file with the following content:

```js
define(
    [
        'uiComponent',
        'Magento_Checkout/js/model/payment/additional-validators',
        '<your_module>/js/model/your-validator'
    ],
    function (Component, additionalValidators, yourValidator) {
        'use strict';
        additionalValidators.registerValidator(yourValidator);
        return Component.extend({});
    }
);
```

## Step 3: Declare the validation in the checkout layout

In your custom module directory, create a new `<your_module_dir>/view/frontend/layout/checkout_index_index.xml` file.

In this file, add the following:

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
<referenceBlock name="checkout.root">
    <arguments>
        <argument name="jsLayout" xsi:type="array">
            <item name="components" xsi:type="array">
                <item name="checkout" xsi:type="array">
                    <item name="children" xsi:type="array">
                        <item name="steps" xsi:type="array">
                            <item name="children" xsi:type="array">
                                <item name="billing-step" xsi:type="array">
                                    <item name="children" xsi:type="array">
                                        <item name="payment" xsi:type="array">
                                            <item name="children" xsi:type="array">
                                                <item name="additional-payment-validators" xsi:type="array">
                                                    <item name="children" xsi:type="array">
                                                        <!-- Declare your validation. START -->
                                                        <item name="your-validator" xsi:type="array">
                                                            <item name="component" xsi:type="string">%your_module_dir%/js/view/%your-validation%</item>
                                                        </item>
                                                        <!-- Declare your validation. END -->
                                                    </item>
                                                </item>
                                            </item>
                                        </item>
                                    </item>
                                </item>
                            </item>
                        </item>
                    </item>
                </item>
            </item>
        </argument>
    </arguments>
</referenceBlock>
    </body>
</page>
```

## Step 4: Deploy static content and clean cache

<InlineAlert variant="info" slots="text"/>

These commands are for production mode. They are not necessary when in developer mode.

1. Deploy static content:

   ```bash
   bin/magento setup:static-content:deploy
   ```

1. Clean cache:

   ```bash
   bin/magento cache:clean
   ```
