# Secure Fields \(iframes\)

Secure Fields allow you to securely collect credit, debit and virtual cards as well as bank account data by injecting iframes to your DOM. A separate iframe for all sensitive values is used. Thereby, the data never touches your server and allows you to capture all other related payment data such as cardholder name, expiry date, etc. directly by yourself.

{% hint style="success" %}
**Using Secure Fields qualifies you for SAQ A.**
{% endhint %}

Browser compatibility for Secure Fields:

| **Browser** | **Version** |
| :--- | :--- |
| Chrome \| Chrome Mobile | &gt;=28 \| &gt;=28 |
| Firefox \| Firefox Mobile | &gt;=31 \| &gt;=31 |
| Internet Explorer \| Internet Explorer Mobile | &gt;=9 \| &gt;= 9 |
| Safari \| Safari Mobile | &gt;=6 \| &gt;=6 |
| Opera \| Opera Mobile | &gt;=24 \| &gt;=22 |
| Blackberry Browser | &gt;=8 |
| Android Browser | &gt;=4 |

## 1. Setup Secure Fields

To get started include the following script on your page. 

{% tabs %}
{% tab title="Secure Fields Script" %}
```javascript
<script src="https://pay.sandbox.datatrans.com/upp/payment/js/secure-fields-2.0.0.js"></script>
```
{% endtab %}

{% tab title="Minified Version" %}
```javascript
<script src="https://pay.sandbox.datatrans.com/upp/payment/js/secure-fields-2.0.0.min.js"></script>
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Please make sure to always load it directly from [https://pay.sandbox.datatrans.com](https://pay.sandbox.datatrans.com)
{% endhint %}

## 2. Create payment form

In order for Secure Fields to insert card number and CVV iframes at the right place, create empty DOM elements and assign them unique IDs. In the example below those are

* `card-number-placeholder`
* `cvv-placeholder`

For the bank account example they are

* `iban-placeholder`
* `account-number-placeholder`
* `branch-number-placeholder`

{% tabs %}
{% tab title="Card number / CVV" %}
```markup
<form>
    <div>
        <div>
            <label for="card-number-placeholder">Card Number</label>
            <!-- card number container -->
            <div id="card-number-placeholder" style="width: 250px;"></div>
        </div>
        <div>
            <label for="cvv-placeholder">Cvv</label>
            <!-- cvv container -->
            <div id="cvv-placeholder" style="width: 90px;"></div>
        </div>

        <button type="button" id="go">Get Token!</button>
    </div>
</form>
```
{% endtab %}

{% tab title="IBAN " %}
```javascript
<form>
    <div>
        <div>
            <label for="iban-placeholder">IBAN</label>
            <!-- IBAN container -->
            <div id="iban-placeholder" style="width: 250px;"></div>
        </div>

        <button type="button" id="go">Get Token!</button>
    </div>
</form>
```
{% endtab %}

{% tab title="Account number / Branch code" %}
```javascript
<form>
    <div>
        <div>
            <label for="account-number-placeholder">account number</label>
            <!-- account number container -->
            <div id="account-number-placeholder" style="width: 90px;"></div>
        </div>
        <div>
            <label for="branch-code-placeholder">branch code</label>
            <!-- branch code container -->
            <div id="branch-code-placeholder" style="width: 90px;"></div>
        </div>

        <button type="button" id="go">Get Token!</button>
    </div>
</form>
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
In test mode, only [test credentials](../../test-card-data.md) are allowed.
{% endhint %}

## 3. Retrieve a Transaction ID

Start with creating a new Secure Fields instance:

```javascript
var secureFields = new SecureFields();

// Multiple instances might be created and used independently:
// e.g. var secureFields2 = new SecureFields();
```

Initialize it with your `merchantId` and specify which DOM element containers should be used to inject the iframes:

{% tabs %}
{% tab title="Card number / CVV" %}
```javascript
secureFields.initTokenize( "1100007006", {
  cardNumber: "card-number-placeholder", 
  cvv: "cvv-placeholder"                
});
```
{% endtab %}

{% tab title="IBAN" %}
```javascript
secureFields.initTokenize( "1100007006", {
  iban: "iban-placeholder"               
});
```
{% endtab %}

{% tab title="Account number / Branch code" %}
```javascript
secureFields.initTokenize( "1100007006", {
  accountNumber: "account-number-placeholder",
  branchCode: "branch-code-placeholder"               
});
```
{% endtab %}
{% endtabs %}

Afterwards submit the form and listen for the success event:

```javascript
$(function() {
  $("#go").click( function() {
    secureFields.submit(); // submit the "form"
  })
});

secureFields.on("success", function(data) {
  if(data.transactionId) {
    // transmit data.transactionId and the rest
    // of the form to your server    
  }
});
```

Instances can be destroyed if no longer used:

```javascript
secureFields.destroy();
```

## 4. Obtain tokens

Once you've transmitted the `transactionId` to your server \(together with the the rest of your form\) you have to execute a **server to server** `GET Token` request to retrieve the tokenized card or bank account values. 

{% hint style="danger" %}
This service requires HTTP basic authentication. The required credentials can be found in our dashboard. Please refer to [API authentication data](../../guides/pci-proxy-dashboard/api-authentication-data.md#basic-authentication) for more information. 
{% endhint %}

{% api-method method="get" host="https://api.sandbox.datatrans.com/upp/services" path="/v1/inline/token" %}
{% api-method-summary %}
Token
{% endapi-method-summary %}

{% api-method-description %}
This endpoint returns the tokenized card or bank account data corresponding to the `transactionId`.  
Please note that this is a **server to server** API call and cannot be called from the browser directly.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Basic MTEwMDAwNzAwNjpLNnFYMXUkIQ==
{% endapi-method-parameter %}
{% endapi-method-headers %}

{% api-method-query-parameters %}
{% api-method-parameter name="transactionId" type="integer" required=true %}
The transactionId obtained via the `secureFields.submit()`operation.
{% endapi-method-parameter %}

{% api-method-parameter name="returnPaymentMethod" type="boolean" required=false %}
Returns payment method used with transaction.
{% endapi-method-parameter %}

{% api-method-parameter name="mandatoryAliasCVV" type="boolean" required=false %}
Wheter the case should be gluten-free or not.
{% endapi-method-parameter %}

{% api-method-parameter name="returnCardInfo" type="boolean" %}
Returns cardInfo object
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
  "aliasCC": "AAABcHxr-sDssdexyrAAAfyXWIgaAF40",
  "aliasCVV": "mVHJkLRrRX-vb9uUzEM40RUN",
  "maskedCard": "424242xxxxxx4242"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Token API calls examples

{% tabs %}
{% tab title="Request" %}
```bash
$ curl "https://api.sandbox.datatrans.com/upp/services/v1/inline/token?transactionId=170419151426624571" \ 
        -u 'merchantId:password'
```
{% endtab %}

{% tab title="Response Card" %}
```bash
{
  "aliasCC": "AAABeM8yJsbssdexyrAAAXnn_sIdAKe0",
  "fingerprint": "F-dV5V8dE0SZLoTurWbq2HZp",
  "aliasCVV": "mVHJkLRrRX-vb9uUzEM40RUN",
  "maskedCard": "424242xxxxxx4242"
}
```
{% endtab %}

{% tab title="Response IBAN" %}
```javascript
{
    "aliasIban": "AAABeKaD2UbssdexyrAAAUN24QvOZg3n",
    "maskedIban": "DE85xxxxxxxxxxxxxx2345"
}
```
{% endtab %}

{% tab title="Response Account number & Branch code" %}
```javascript
{
    "aliasAccountNumber": "AAABeKahwGDssdexyrAAAV8w_R0dlq9b",
    "maskedAccountNumber": "xxxx0604",
    "aliasBranchCode": "AAABeKahwGDssdexyrAAAadGCQEwl6MZ"
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Example: returnPaymentMethod=true returning paymentMethod" %}
```bash
$ curl "https://api.sandbox.datatrans.com/upp/services/v1/inline/token?transactionId=170419151426624571&returnPaymentMethod=true" \
       -u 'merchantId:password'
```
{% endtab %}

{% tab title="Response" %}
```bash
{
  "aliasCC": "AAABcHxr-sDssdexyrAAAfyXWIgaAF40",
  "aliasCVV": "mVHJkLRrRX-vb9uUzEM40RUN",
  "paymentMethod": "VIS",
  "maskedCard": "424242xxxxxx4242"
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Example: mandatoryAliasCVV=true w/ transactionId that has no CVV token" %}
```bash
$ curl "https://api.sandbox.datatrans.com/upp/services/v1/inline/token?transactionId=170822090245534063&mandatoryAliasCVV=true" \
       -u 'merchantId:password'
```
{% endtab %}

{% tab title="Response" %}
```bash
400 Bad Request
Tokenization with CVV not found
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Example: returnCardInfo=true returning CardInfo " %}
```javascript
curl -L -X GET 'https://api.sandbox.datatrans.com/upp/services/v1/inline/token?transactionId=210202095141262032&returnCardInfo=true' \
-H 'merchantId:password'
```
{% endtab %}

{% tab title="Response" %}
```javascript
{
    "aliasCC": "AAABeM8yJsbssdexyrAAAXnn_sIdAKe0",
    "fingerprint": "F-dV5V8dE0SZLoTurWbq2HZp",
    "aliasCVV": "ScRuEmNjRJ682mIGKHA9xx_R",
    "maskedCard": "424242xxxxxx4242",
    "cardInfo": {
        "brand": "VISA CREDIT",
        "type": "credit",
        "usage": "consumer",
        "country": "GB",
        "issuer": "DATATRANS"
    }
}
```
{% endtab %}
{% endtabs %}

### Error table 

| Error message | Cause / Explanation |
| :--- | :--- |
| Tokenization expired | The `transactionId` has expired. Please note that it is valid for 30 minutes only. |
| Tokenization not found | The merchant id used for the transaction id creation does not match the merchant id used for the GET Token call. |

## Examples

Please visit our GitHub [repository](https://github.com/datatrans/secure-fields-sample) for additional examples of integration versions:   
  
Demo Basic: [https://datatrans.github.io/secure-fields-sample/](https://datatrans.github.io/secure-fields-sample/index.html)  
Demo with horizontal fields: [https://datatrans.github.io/secure-fields-sample/inline-example.html](https://github.com/datatrans/secure-fields-sample/blob/master/inline-example.html)  
Demo with floating labels: [https://github.com/datatrans/secure-fields-sample/blob/master/floating-label.html](https://github.com/datatrans/secure-fields-sample/blob/master/floating-label.html)

An example of how to implement this behaviour in modern web applications can be found [here](https://github.com/datatrans/secure-fields-sample/tree/master/react-example)

{% hint style="warning" %}
In test mode, only [test credentials](../../test-card-data.md) are allowed.
{% endhint %}

Please also have a look at [Styling](initialization-and-styling.md) and [Events](events.md) references.

## Next up

{% page-ref page="../../use-stored-cards/" %}



