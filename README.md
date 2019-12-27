## React + Stripe Ecommerce Store Template

Artisanal hand-rolled e-commerce site.

Written in React, served up with Express, and integrated with [Stripe Dashboard](https://stripe.com/us/payments). 

Uses [Material-UI](http://material-ui.com/) and [styled-components](https://www.styled-components.com/) for the design. 

Site also includes a password-protected admin view, Nodemailer integration for sending order updates, and email templates built with Handlebars.

### Demo
[Demo of store](https://www.gpxjewelry.com/) - with some extra visualization magic on top :)

### Quickstart
```
yarn install
yarn start
```
This will open a browser tab with the store running. The config file in `/src/assets/` will be running the store, and at first you should see one product. The `$Infinity - $-Infinity` price tag is notifying us that there is an error connecting to the Stripe account, which we hope would be true since we haven't set a stripe key or a stripe product ID yet!

### Store Config Philosophy

Individual stores are created via a config file. There are three example configs in `/src/assets/`. The config generally looks like:
```
{
  "store_name": "The best little ecommerce site in Texas",
  "store_slug": "react-stripe-store",
  "api_key": {STRIPE_PUBLIC_KEY},
  "colors": {
    "primary": {
      "main": "#FE8A00",
      "dark": "#FD7300",
      "contrastText": "#FFF"
    },
    "secondary": {
      "main": "#00FFB4"
    }
  },
  "products": [
    {
      "name": "Super Cool Product",
      "url": "url-for-product",
      "stripe_id": {STRIPE_PRODUCT_ID},
      "description": "What a great product!",
      "photos": ["photo1.jpg","photo2.jpg"],
      "details": [
        "These are details that get rendered as bullet points",
        "Useful for short + sweet info"
      ]
    },
  ...
  ]
}
```

Each product can also have an optional `variants` key for additional metadata to be saved for each product, rendered as a dropdown. This is for saving options without having to create individual SKUs in Stripe.
```
"variants": {
  "name": "metadata",
  "options":[
    {"label": "option 1"},
    {"label": "option 2"}
  ]
}
```

Items in that config in all caps are sourced from Stripe. This project makes use of Stripe Dashboard to keep track of Product inventories, and SKUs (this allows Stripe to handle all payment info, reducing the risk of man-in-the-middle issues). On loading a product page, this site will ask Stripe for the SKUs associated with the given product ID.

Items added to the cart are saved via `localStorage`, which namespaces them according to the `store_slug`, such that you can run several stores at once and keep each purchase separate.

Images are expected to live in `/public/photos/{product.url}/{product.photos.name}`. The site will also add a CSS class on the body that is the store slug, for store-specific CSS.

### Switching Between Stores
If you are developing multiple stores at once:
* Switch config files in `App.js` in line `import config from './assets/{YOUR_CONFIG_FILE}'`
* Also in `App.js`, change the `Landing` componet to your particular file: `./components/Landing-{STORE_SLUG}`
* Change the Stripe Secret key in your `config.env` file

### Secret Sauce

Orders can be tracked at `/admin`, which is accessed via `/login`. The admin password is saved as an env variable, and admin status is saved in a session cookie.

Secrets are stored via environment variables, which are created via [dotenv](https://www.npmjs.com/package/dotenv). This process expects a file titled `config.env` in your `/root` folder, with the following items:
```
STRIPE_KEY={get this from stripe}
ADMIN_PW={your password to log in}
SESSION_SECRET={generate a random string here, make it hard to guess}
EMAIL_FROM=
EMAIL_CLIENT_ID=
EMAIL_CLIENT_SECRET=
EMAIL_REFRESH_TOKEN=
EMAIL_ACCESS_TOKEN=
EMAIL_EXPIRES=
```

All of the `EMAIL_` items are generated by the following [Gmail Oauth2 Setup](https://stackoverflow.com/a/43202668).

### Email Templates

The repo comes with multiple order templates, which are triggered by updating the order status on the `/admin` page. Here's an example email, with all of the relevant personalizations (store name, banner color) coming from the config file.

!["Order Confirmation"](./email_example.png?raw=true "Order Confirmation")

### Deployment

YMMV but I used this [tutorial](https://hackernoon.com/start-to-finish-deploying-a-react-app-on-digitalocean-bcfae9e6d01b) for a deployment on DigitalOcean, up until the last half of the last step (AKA the `default` file config). (Also [yarn install](https://stackoverflow.com/questions/42606941/install-yarn-ubuntu-16-04-linux-mint-18-1))

Remember to set up the `config.env` file!

Daeomonize step: `pm2 start server/index.js`

Next was setting up nginx according to [this guide](https://www.digitalocean.com/community/questions/how-do-i-point-my-custom-domain-to-my-ip-port-41-111-20-36-8080), ex:
```
server {
  listen 80;
  server_name myapp.domain.com;

  location / {
    proxy_pass http://localhost:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

Then using [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-with-nginx-server-blocks-on-ubuntu-16-04) for setting up an SSL cert with Let's Encrypt. And then adding port 22 for ssh a la [this fix](https://www.digitalocean.com/community/questions/i-tried-to-ssh-root-ipaddress-i-received-error-port-22-connection-refused).

Deploy script:
```
ssh {username}@{IP address}
cd {project}
git pull
yarn run build
pm2 restart all
```

### Conclusion

Don't @ me :)