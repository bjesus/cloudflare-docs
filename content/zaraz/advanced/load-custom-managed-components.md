---
pcx_content_type: how-to
title: Custom Managed Components
weight: 15
---

# Loading a Custom Managed Component

Zaraz supports loading custom third-party tools using the [Managed Components](https://managedcomponents.dev/) format. These can be Managed Components that you have developed yourself, or that were developed by others. For security reasons, using custom Managed Components with Zaraz is done by converting them into a Cloudflare Worker running in your account.

If you are new to Managed Components, we recommend getting started by [creating a Managed Component](https://managedcomponents.dev/getting-started/quickstart) or checking out a [demo Managed Component](https://github.com/managed-components/demo).

{{<Aside type="note">}}
Custom Managed Components are only available for accounts on a [Workers Paid plan](/workers/platform/pricing/).
{{</Aside>}}

## Prepare a Managed Component

If your Managed Component requires building, transpiling or bundling, you need to do it before you can convert it to a Cloudflare Worker. This is usually done by running `npm run build` or an equivalent. You need have a JavaScript file that exports the default Managed Component function for your Managed Component to be ready for deployment.

In this guide, we will use a simple example of a Managed Component that counts the users pageviews and logs it in the console:

```javascript
// File: index.js
export default async function (manager) {
  // Add a pageview event
  manager.addEventListener("pageview", event, () => {
    const { client } = event;

    // Get the variable "counter" from the client's cookies and increase by 1
    let counter = parseInt(client.get("counter")) || 0;
    counter += 1;

    // Log the increased number
    client.execute(`console.log('Views: ${counter}')`);

    // Store the increased number for the next visit
    client.set("counter", counter);
  });
}
```

## Deploy a Managed Component to Cloudflare

1. Open a terminal in your managed component’s root directory.
2. From there, run `npx managed-component-to-cloudflare-worker ./index.js my-new-counter-mc`, which will deploy the Managed Component to a Cloudflare Worker. Change `./index.js` to match the path of your Managed Component JavaScript file, and change the name of the component to your liking.
3. Your Managed Component should now be [visible on your account](https://dash.cloudflare.com/redirect?account=/workers-and-pages) as a Cloudflare Worker prefixed with `custom-mc`.

## Configure a Managed Component in Cloudflare

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/login) and select your account and domain.
2. Select **Zaraz** > **Tools Configuration** > [**Third-party tools**](https://dash.cloudflare.com/?to=/:account/:zone/zaraz/tools-config/tools/catalog).
3. Select **Add new tool** and choose **Custom Managed Component** from the tools library page. Select **Continue** to confirm your selection.
4. In **Select Custom MC**, choose a Custom Managed Component that you have deployed to your account, such as `my-new-counter-mc`. Select **Continue**.
5. In **Grant Permissions**, select the permissions you want to grant the Custom Managed Component. If the Managed Component was developed by someone you do not trust, pay close attention to what permissions you grant it. Select **Continue**.
6. In **Set up**, configure the settings for your new tool. The information you need to enter will depend on the tool your Managed Component code. You can add Settings and Default fields.
7. Select **Save**.

## Adding Actions

While your tool is now configured, it does not have any actions associated with it yet. Adding new actions will tell Zaraz when to contact your Managed Component, and what information to give it. When adding the actions, make sure to verify the Action Type you are using. The types `pageview` and `event` are the most commonly used ones, but you can add any action type to match any event listener your Managed Component is using. Refer to [Create actions](/zaraz/get-started/create-actions/) to learn how to create additional actions.

If your Managed Component listens to `ecommerce` events, toggle **E-commerce tracking** in the Managed Component Settings page.

## Unsupported Features

As of now, Custom Managed Components do not support the use of the following methods yet:

- `manager.registerEmbed`
- `manager.registerWidget`
- `manager.proxy`
- `manager.route`
- `manager.serve`
- `manager.get`
- `manager.set`
- `manager.useCache`
- `manager.invalidateCache`