# Lab: Create the Foundation

In this portion of the port to Ionic v4 we will build a solid foundation upon which to build. This foundation will start with a new Ionic v4 application. We will then add the libraries and plugins that we use and we will rename the pages generated by our starter to match the v3 application.

This part of the process may be a bit tedioius is spots, but remember that this is building the foundation upon which the rest of our port will take place, so it is very important to do this part with care.

## Generate a Starting Application

The file system layout and build tooling of an Ionic v4 application matches that of an Angular CLI application and not that of an Ioinc v3 application. Therefore the best way to handle this is to create a whole new application and copy your code over once chunk at a time.

`ionic start ionic-weather-v4 tabs --type=angular --cordova`

1. Answer **Y** (yes) to the Ionic Pro SDK question
1. You will be asked if you want to link to an existing app or create a new one
    - Choose to **Link an existing app on Ionic Pro**
1. You will be asked if you would like to use the GitHub integration or Ionic Pro
    - Choose **Ionic Pro**

## Add Libraries

Our v3 application used a few libraries and plugins. Some required configuration. Let's install those now and do the configuration that is required.

1. copy the `config.xml` file from the v3 application, we can use this as-is
1. make a `src/assets/images` folder and copy the images from the folder with the same name in the v3 app
1. copy the `icon.png` and `splash.png` from the v3 app's `resources` folder to the v4 app's `resources` folder
1. `npm i @ionic/storage`
1. `ionic cordova plugin add cordova-plugin-geolocation --variable GEOLOCATION_USAGE_DESCRIPTION="To determine the location for which to get weather data"`
1. `npm i @ionic-native/geolocation@beta`
1. `npm i kws-weather-widgets`
1. edit the `package.json` file
   1. update the `@ionic-native` libraries to the same beta version
   1. update the `@ionic/angular` library to the latest beta
   1. copy the `cordova-plugin-ionic` entries from the v3 project (this is in two places)
1. `npm i` - this will install/update everythng that was just changed
1. `ionic cordova platform add ios`
1. `ionic cordova platform add android`

Those last two steps take a while, so while those are running we can work on the configuration a bit:

**src/app/app.module.ts**

* import `HttpClientModule`, `Pro`, `IonicStorageModule`, and `Geolocation` (note the `ngx` at the end of the path)
* initialize Pro
* add `HttpClientModule` and `IonicStorageModule.forRoot()` to the `imports` section
* add `Geolocation` to the `providers` section

```TypeScript
import { HttpClientModule } from '@angular/common/http';

...
import { Pro } from '@ionic/pro';
...

import { IonicStorageModule } from '@ionic/storage';
import { Geolocation } from '@ionic-native/geolocation/ngx';

...

Pro.init('1ec81629', {
  appVersion: '0.0.1'
});


@NgModule({
  declarations: [AppComponent, UserPreferencesComponent],
  entryComponents: [],
  imports: [
    BrowserModule,
    HttpClientModule,
    IonicModule.forRoot(),
    IonicStorageModule.forRoot(),
    AppRoutingModule
  ],
  providers: [
    Geolocation,
    StatusBar,
    SplashScreen,
    { provide: RouteReuseStrategy, useClass: IonicRouteStrategy }
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

**src/app/home/home.module.ts, etc**

The three child pages of the application (not the tabs page) will all use custom elements from `kws-weather-widgets`, so they all need to define the `CUSTOM_ELEMENTS_SCHEMA`.  Here is an example:

```TypeScript
import { IonicModule } from '@ionic/angular';
import { RouterModule } from '@angular/router';
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { HomePage } from './home.page';

@NgModule({
  imports: [
    IonicModule,
    CommonModule,
    FormsModule,
    RouterModule.forChild([{ path: '', component: HomePage }])
  ],
  declarations: [HomePage],
  schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class HomePageModule {}
```

**Commit Here!**

At this point, you should still have a running application. Check that this is true.

This is a really good place to do a commit. Note that we aren't _really_ going to save a commit history from this or anything. This is just so we have a fallback point if we mess anything up. This won't be pushed anywhere or shared with anyone, so go ahead and commit to master.

We can discuss strategies for doing a port as a team later.

```bash
git add .
git commit -am "install libraries"
```

## Rename Pages 

Renaming the source can be tedious, so I suggest creating a simple script to perform the actions instead. I find this much easier than doing it manually. Here is the script I use:

```bash
cd src/app

mkdir current-weather/ forecast/ uv-index/

git mv home/home.module.ts current-weather/current-weather.module.ts
git mv home/home.page.scss current-weather/current-weather.page.scss
git mv home/home.page.ts current-weather/current-weather.page.ts
git mv home/home.page.html current-weather/current-weather.page.html
git mv home/home.page.spec.ts current-weather/current-weather.page.spec.ts

git mv about/about.module.ts forecast/forecast.module.ts
git mv about/about.page.html forecast/forecast.page.html
git mv about/about.page.scss forecast/forecast.page.scss
git mv about/about.page.spec.ts forecast/forecast.page.spec.ts
git mv about/about.page.ts forecast/forecast.page.ts

git mv contact/contact.module.ts uv-index/uv-index.module.ts
git mv contact/contact.page.html uv-index/uv-index.page.html
git mv contact/contact.page.scss uv-index/uv-index.page.scss
git mv contact/contact.page.spec.ts uv-index/uv-index.page.spec.ts
git mv contact/contact.page.ts uv-index/uv-index.page.ts

rmdir home/ about/ contact
```

At this point, your application should be totally broken. We will use your editor's tools to fix that. My example assumes VSCode, but most editors have similar features you can use.

Use your search tool to find bad path specifications and fix them. Do not change any selction tags or class names at this time:

1. search for "home.page" and change "home" to "current-weather"
1. search for "home.module" and change "home" to "current-weather"
1. search for "about.page" and change "about" to "forecast"
1. search for "about.module" and change "about" to "forecast"
1. search for "contact.page" and change "contact" to "uv-index"
1. search for "contact.module" and change "contact" to "uv-index"

Your code _should_ be working again.

## Rename Classes and Modules

Open the class file for each of the three renamed pages. Do the following:

1. use the "Rename Symbol" tool from VSCode to rename the classes:
   - HomePage -> CurrentWeatherPage
   - AboutPage -> ForecastPage
   - ContactPage -> UVIndexPage
1. change the `selctor` for each page

Perform a similar action on the `module.ts` files to rename the pages' modules.

## Change the Icons and Labels

Have a look at the tabs page HTML in your v3 application. Use that as a guide to change the names of the icons and labels in your v4 application.

## Change the Routing and Outlet Names

Notice that the tabs starter for v4 uses Angular's router outlines, and that we have our own derivation of the outlet called `ion-router-outlet`. Both the outlet name and the path end up in the URL, and they do not currently match the names of the tabs. Let's fix this.

* in `tabs.page.html`
   * change the outlet names
   * change the hrefs
* in `tabs.router.module.ts`
   * change the `path` specifications
   * change the `outlet` specifications
   * change the `redirectTo` paths

We now have a nice, solid foundation. Another commit is in order.