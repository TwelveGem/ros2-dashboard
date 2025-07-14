# ROS2-Dashbaord

This repository hosts a webserver that connected to ROS2 through websockets. The default address used is `ws://localhost:9090`. If this is not the correct url for your usecase, change the variable `rosUrl` within this [file](src/routes/+page.svelte).

## Running

In order to start the server, one of the following commands are run

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## How to Use

On startup, there will be a few pre-populated topics. In order to edit, click anywhere on the card and a menu will pop up. The topic path and type can be modified. If the topic you are looking for is not in the list, a custom type can be selected.

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
