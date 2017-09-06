# Payment Page

The [Saferpay Payment Page](https://saferpay.github.io/jsonapi/#ChapterPaymentPage) can be used both with a Saferpay eCommerce contract and with a Saferpay business contract. It allows the processing of all payment methods available through Saferpay. Once integrated, more payment methods can also be activated at any time and without major adjustments.

test test

## Description of the General Process for Using PaymentPage

### <a name="pp-initialize"></a> PaymentPage Initialize

The process begins with the [PaymentPage Initialize](https://saferpay.github.io/jsonapi/#Payment_v1_PaymentPage_Initialize) request. With this request, you forward all data necessary for the payment to Saferpay. These include the customer number (CustomerId), the terminal number (Terminal Id), the currency (CurrencyCode), the amount (Value), the internal reference no. of the merchant system (OrderId), and the return addresses (ReturnUrls) to which the customer will return after leaving the payment page.

### Information on the Use and Significance of the Available Parameters

+ **PaymentMethods:** By default, the payment page will always show all payment methods approved for the terminal in question. Limiting the display to a single item or preselection of the payment methods in the shop can be achieved via the **PaymentMethods** parameter. When using this parameter, the only payment methods displayed are those whose values are forwarded.  When forwarding multiple values, the PaymentPage opens with a page which offers the option to select the appropriate payment method. If only one value is passed, the PaymentPage skips the selection window. Invalid values or values from payment methods which are not available on the terminal or not available with the specified currency are ignored by the PaymentPage. For example, if only one value is forwarded and this is invalid, the Payment Page will display the same options as if **PaymentMethods** had not been used.

+ **ReturnUrls:** Via the **ReturnUrls**, the customer is returned to the shop after the transaction. For security reasons, Saferpay returns no data to return addresses. The identification of the payment or the returning customers is up to the merchant. We recommend using your own parameters. These can be attached via HTTP GET to the **ReturnUrls**. When a ReturnUrl is called, Saferpay returns the appended parameter, thus enabling identification of the customer/transaction. The ReturnUrls are **ALWAYS** called through the clients Browser. 

+ **NotifyUrl:** Although it is entirely optional, we strongly recommend integrating the **NotifyUrl** (see Notification container). With **NotifyUrl**, a notification of the shop in the event of a successful payment is made via a http-GET, regardless of connection problems that may prevent the customer getting back to the **SuccessUrl**. Without **NotifyUrl**, such an authorisation would be present in Saferpay Backoffice, however the shop wouldn’t have received any feedback. Technically speaking, **NotifyUrl** notifies the merchant system, as a redundant measure. In contrast to the **Success URL**, **NotifyUrl** is called up directly via HTTP GET by the Saferpay servers upon successful payment and not the client. It should be noted that the notification via **NotifyUrl** takes place **in addition to the Success URL**. Moreover, Saferpay expects the merchant server to answer the call with a HTTP status code 200(OK). If no (e.g. a Timeout), or a different status code (e.g. http 500 ) is received, Saferpay will call the **NotifyUrl** up to two times more, for a total of three times, to ensure, that the previous error is not caused by a temporal problem. Note, that the merchants application needs to differentiate between the **SuccessUrl** and **NotifyUrl**, to prevent the application from processing the same transaction two, or more times. Finally, it should also be noted, that the **NotifyUrl** also does not return any data to the merchants application (See **ReturnUrls** above!)!

+ **BillingAddressForm and DeliveryAddressForm:** The use of **BillingAddressForm** and/or **DeliveryAddressForm** allows card holders to fill in an address-form during payment. This enables merchants to capture their address-data without having to implement a form on their own. It can be freely decided which entry fields are mandatory to fill in and which are not. Upon completion of the payment, Saferpay returns the addresses via the [PaymentPage Assert](https://saferpay.github.io/jsonapi/#Payment_v1_PaymentPage_Assert) response.

+ **RegisterAlias:** Via the **RegisterAlias** container, card details from a payment can be stored safely and in conformity with PCI. For this, the alias for the card details is transmitted back to the merchant system with the authorisation response. It is then available for another purchase in the shop, without customers having to enter their card details again. See for example the [Initialize request above](https://saferpay.github.io/sndbx/Integration_trx.html#trx-ini). The parameter **IdGenerator** is used to determine how the alias is generated. If **MANUAL** is specified, the shop system passes an absolutely unique value. With **RANDOM**, a random alphanumeric value is generated by Saferpay. With **RANDOM_UNIQUE**, it will be additionally verified prior to generation of the alias, as to whether or not the card number/expiration date combination already exists as an alias entry in the database. If so, the already existing alias is returned and nothing new is generated.
>
><i class="glyphicon glyphicon-hand-right"></i> **NOTE:** A card will be registered only after a successful authorisation. Also, the PaymentPage may be able to register a card, it is **NOT** able to use them for future payments. Those may only be made via the [TransactionInterface](https://saferpay.github.io/sndbx/Integration_trx.html).
>

## <a name="pp-initialize-response"></a>PaymentPage Initialize Response

### Inside the PaymentPage Initialize Response, the following parameters are important:

+ **Token:** The **Token** refers to the values temporarily stored regarding the Saferpay transaction and is mandatory during subsequent processing of the transaction ([for more information, see e.g. PaymentPage Assert](http://saferpay.github.io/jsonapi/index.html#Payment_v1_PaymentPage_Assert)). The Token should be coupled to the http-GET parameters that were previously attached to the **ReturnUrls** and **NotifyUr**l for identification. 

+ **RedirectUrls:** The **RedirectURLs** provides the address via which the buyer is redirected to the PaymentPage. This can be done automatically via calling up the Iframe or by embedding the URL in an HTML link that must be clicked on by the buyer.

## <a name="pp-transaction"></a> Redirect and Transaction

The transaction will be fully processed by the PaymentPage. The Payment Page handles all steps automatically, including [3D Secure](https://saferpay.github.io/sndbx/index.html#3ds) and [DCC](https://saferpay.github.io/sndbx/index.html#dcc). No additional steps are necessary on the merchant website.

## <a name="pp-retshop"></a> Return to the Shop

Once the transaction is complete, the card holder – depending on the outcome – is taken back to the shop, to one of the **ReturnUrls**. Here, the GET parameters can be read and the **Token** can be assigned to the transaction. With the **Token**, the payment can be continued to the next step.

## <a name="pp-assert"></a> PaymentPage Assert

With the [PaymentPage Assert](https://saferpay.github.io/jsonapi/#Payment_v1_PaymentPage_Assert), the results of a transaction are requested. The returned data may be stored on the merchant side.

Based on the data obtained, it is to be decided whether or not a transaction is to be further processed. The following data is interesting in this regard:

+ **Transaction > ID:** The transaction identifier returned in the container **Transaction** with **Id** is a unique identifier for a transaction. The value is obligatory for further processing steps (Transaction Capture or Transaction Cancel) and should therefore be saved.

+ **ThreeDs:** This container provides information about whether or notliability shift through [3D Secure](https://saferpay.github.io/sndbx/index.html#3ds) is available or not. It is up to merchant whether or not he wants to accept transactions without liability shift. Evaluation of the parameter provides the opportunity for merchants to incorporate appropriate rules here.
>
><i class="glyphicon glyphicon-hand-right"></i> **IMPORTANT:** Accepting transaction without LiabilityShift happens at the merchants own risk.
>

+ **Transaction > Status:** As already described [here](https://saferpay.github.io/sndbx/General.html#capture-batch), this status states whether or not a transaction has to be finalized via [Capture](https://saferpay.github.io/jsonapi/#Payment_v1_Transaction_Capture). If this status is not **CAPTURED**, the capture must be executed in order to finalise the transaction.

>
><i class="glyphicon glyphicon-hand-right"></i> **Tipp:** You can also call the Assert, if the FailUrl has been called. It will then give you information about the error!
>

## <a name="pp-captcancel"></a> Capture or Cancel

Subsequently, the transaction will be finalised via [**Capture**](https://saferpay.github.io/jsonapi/#Payment_v1_Transaction_Capture) or broken off via [**Cancel**](https://saferpay.github.io/jsonapi/#Payment_v1_Transaction_Cancel).For this, the transaction identifier **Id** is required. Please refer to the notes on the payment methods on if and when a **Capture** is necessary, and whether a **Cancel** can still be carried out.

Once these steps are complete, the transaction is completed.

