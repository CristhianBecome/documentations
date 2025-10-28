# Implementation

## Recommended Usage

The recommended way to use this SDK is by integrating the script from `https://identity.svi.becomedigital.net/resources/button.js` into your code.

### Integration Steps

- Add the Script: Include the following script in your HTML <head> (recommended) or before the closing <body> tag:

```html
<script src="https://identity.svi.becomedigital.net/sdk/button.js"></script>
```

## Usage in Your Code

To include the Become KYC button in your website, simply add the following HTML tag to your page:

```html
<become-button
  userId="<YOUR_USER_ID>"
  contractId="<YOUR_CONTRACT_ID>"
  token="<YOUR_JWT_TOKEN>"
/>
```

How It Looks on Your Page:

<img src="https://gist.githubusercontent.com/Tyg0th/15c5131ef7d2b24b9effa97eb45dedce/raw/07a5e1f3e428bd1d32bfe2940591872e1ae1ec2d/become-button-example.jpg" width="211" />


Alternatively, you can dynamically create the button using JavaScript, as shown below:

```js
const button = document.createElement("become-button");
button.setAttribute("userId", "user_12345");
button.setAttribute("contractId", "contract_67890");
button.setAttribute("token", "your_jwt_token_here");
document.body.appendChild(button);
```

Note:

If you prefer, you can compile the Become-web-button yourself, ensuring that the `signupHost` points to `https://identity.svi.becomedigital.net`.