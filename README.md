# OAuth2 Library 

## Overview <a name="7"></a>

1. [__Features__](#1)
2. [__Setting up a new project with OAuth2__](#2)
3. [__Testing with 'testbench' project__](#3)
4. [__The 'oauth' project and how it works__](#4)
5. [__All methods provided__](#5)
6. [__Support__](#6)


---
### 1. Features <a name="1"></a>

- easy to use 
- makes it faster to develop
- ...

---

### 2. Setting up a new project with OAuth2 <a name="2"></a>

If you want to use this library in your own project, to simplify the process of authorization, just follow these steps.

1. Create a new project.

```
ng new <your_project_name>
```

2. Install the OAuth2 Library.

```
cd <your_project_name>
npm install @beemoos/oauth
```

3. Add a redirect component. This component will basically be used when authorization failed. Our OAuth2Guard checks whether or not you have permission to visit a specific site and navigates us to this component, in case you don't. The component itself handles the login process.

```
ng generate component redirect
ng generate module redirect
```

4. Add routing to your project. Go to your 'app.module' and add the following routes.

```
const appRoutes = [
  {path: '', redirectTo: '/dashboard', pathMatch: 'full', canActivate: [OAuth2Guard]},
  {path: 'dashboard', component: AppComponent, canActivate: [OAuth2Guard]},
  {path: 'redirect', component: RedirectComponent}
];
```
Every route with the ``'canActivate: [OAuth2Guard]'-Tag`` can only be accessed when you are successfully authorized. If you have no permission, you will get redirected to the login form. 

> The components you're referring to can be changed to whatever component you need to.

5. After completing that you have to add some imports in your 'app.module'. Some of them for later use.

```
import { OAuth2Module, OAuth2Guard } from '@beemoos/oauth';
import { RouterModule } from '@angular/router'; 

// Also import all your components you're using. In this case:
import { AppComponent } from './app.component';
import { RedirectComponent } from './redirect/redirect.component';
```

6. You also have to edit the @NgModule annotation. Add all your components to the 'declarations' array and all your imports to the 'imports' array. The interesting thing however is the fact that the import of our OAuth2Module lets us create a config.  You can change all of the given values to your needs. ```'BaseURL, ClientID, ClientSecret, Scopes, RedirectRoute and Production'```
You also have to add the OAuth2Guard to your 'providers' array.

```
@NgModule({
  declarations: [
    // Add all of your components that you're using. In this case:
    AppComponent,
    RedirectComponent
  ],
  imports: [
    // Add all of your imported modules. In this case:
    BrowserModule,
    CommonModule,
    RouterModule.forRoot(
      appRoutes
    ),
    // This part let's you configure the authentication process. Just change the values to your needs:
    OAuth2Module.forRoot({
      oauthBaseUrl: 'http://localhost:19080',
      oauthClientId: 'clientId',
      oauthClientSecret: 'value',
      oauthScope: 'read, write',
      redirectRoute: '/redirect',
      production: false
    })    
  ],
  providers: [ OAuth2Guard ],
  bootstrap: [ AppComponent ]
})
```

Combining everything your 'app.module' should look similar to this:

```
import { CommonModule } from '@angular/common';
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

// Added imports:
import { OAuth2Guard, OAuth2Module } from '@beemoos/oauth';
import { RouterModule } from '@angular/router';
import { AppComponent } from './app.component';
import { RedirectComponent } from './redirect/redirect.component';

// Your routes:
const routes = [
  {path: '', redirectTo: '/dashboard', pathMatch: 'full', canActivate: [ OAuth2Guard ]},
  {path: 'dashboard', component: AppComponent, canActivate: [ OAuth2Guard ]},
  {path: 'redirect', component: RedirectComponent}
];

// Component-Declarations, imported modules and the providers.
@NgModule({
  declarations: [
    AppComponent,
    RedirectComponent
  ],
  imports: [
    BrowserModule,
    CommonModule,
    RouterModule.forRoot( routes ),
    OAuth2Module.forRoot({
      oauthBaseUrl: 'http://localhost:19080',
      oauthClientId: 'test',
      oauthClientSecret: '79hnpw20',
      oauthScope: 'read, write',
      redirectRoute: '/redirect',
      production: false
    })
  ],
  providers: [ OAuth2Guard ],
  bootstrap: [ AppComponent ]
})

export class AppModule { }

```
7. In order for us to use our service we have to inject it first. Go to the component that needs access to the OAuth2 service and inject it. In this example we need the service in our 'AppComponent' and in our 'RedirectComponent'. 
Create a constructor for these components and inject the service into them. You have to add the import statement too.

```
import { OAuth2Service } from '@beemoos/oauth';

...

export class AppComponent { // <-- same for RedirectComponent
    constructor(public service: OAuth2Service) { }
}
```

Also add a ```'<router-outlet></router-outlet>'-Tag``` to your 'app.component.html'. This will load the corresponding HTML for your routes. 

8. After doing so you can use the service. A simple example for logging-in could be as follows: It has do with our previously created ``'RedirectComponent'``. We have to add the logic inside of the component, which will handle the login process. Navigate to our component and add the following. (We also have to inject a Router in order to be able to navigate to other routes, an ActivatedRoute, which allows us to query the URL parameters, and our OAuth2Service.)

```
export class RedirectComponent implements OnInit {

  // Injection of a Router, an ActivatedRoute and our OAuth2Service
  constructor(private router: Router,
              private route:ActivatedRoute,
              private service: OAuth2Service) { }

  ngOnInit() {
      this.route.queryParams.subscribe((params) => {
          // Request an access token, if there is a code available in the query parameters
          if (params['code']) {
              // executing the method 'requestToken(arg)' from our OAuth2Service
              this.service.requestToken(params.code);
          } else if (!this.service.isAuthenticated()) { // <-- Log in, if you're not yet authenticated
              // The method 'openLogin()' handles everything that has to do with the login process
              this.service.openLogin();
          } else {
            if (localStorage.getItem('origin_path') !== null) {
              // Redirect to original path, if it was set
              const redirectPath = localStorage.getItem('origin_path');
              localStorage.removeItem('origin_path');
              this.router.navigateByUrl(redirectPath);
            } else {
              this.router.navigateByUrl('/');
            }
          }
      });
  }
}
```

If you then open the website you will be redirected to your previously specified OAuth Server with the given credentials and are prompted to login. You will then be redirected back and your access token will be stored in the 'localStorage' along with the refresh token. If you want to get information on what scopes you specified or you want a button, which logs you out, you can use methods, which are already usable by the OAuth2Service. For a list of all methods read ```Section 5```.
Realizing a Logout-button would only require a button, which executes a method of your component, which executes the services logout method.

```
// In our 'app.component.html' we add a button thats going to execute 'logoutWhenButtonPressed()'
<button (click)="logoutWhenButtonPressed()">Logout</button>

// In our 'app.component.ts' we execute the method 'logout()', which is included in the library.

export class AppComponent {
    constructor(private service: OAuth2Service) { }

    logoutWhenButtonPressed() {
        this.service.logout();
    }
}
```
---

### 3. Testing with 'testbench' project <a name="3"></a>

1. If you want to test this library you can use the 'testbench' project. Navigate to the folder and install all missing packages.

```
cd testbench/
npm install
npm install @beemoos/oauth
```

2. After that you have to setup your OAuth servers credentials correctly. Navigate to your 'app.module' and change the values to your needs.

```
[...]

@NgModule({
  declarations: [
    AppComponent,
    RedirectComponent
  ],
  imports: [
    BrowserModule,
    CommonModule,
    // Change the values below to your needs. Otherwise you will get errors.
    OAuth2Module.forRoot({
      oauthBaseUrl: 'http://localhost:19080', // your OAuth server
      oauthClientId: '',                      // your ClientId
      oauthClientSecret: '',                  // your ClientSecret
      oauthScope: 'read, write',              // scopes you want to add (seperate multiple with commas.)
      redirectRoute: '/redirect',             // enter your redirect route
      production: false
    }),
    RouterModule.forRoot(
      appRoutes
    )
  ],
  providers: [ OAuth2Guard ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```
3. You can now use the project to test the functionality of methods or similar. 
If you go into the 'app.component.html' you can add a button, text or anything that executes a method in your 'app.component.ts'.

```
// In your 'app.component.html'
<h3>AppComponent<h3>
<button (click)=executeExample()>Example</button>
<p (click)=executeExample()>TextExample</p>
<router-outlet></router-outlet>

// In your 'app.component.ts'
export class AppComponent {
  constructor(private oAuthService: OAuth2Service) { }

  executeExample() {
    // execute any method you want, e.g. 'getAccessToken()'
    console.log(this.oAuthService.getAccessToken());
  }
}
```

You can find a full list of methods under ```Section 5```.

> In order for it to work your OAuth server has to be setup correctly.

---

### 4. The 'oauth' project and how it works <a name="4"></a>

If you want to know how the service itself works you can take a look at the 'oauth' project. Navigate to it and install all missing packages.
```
npm install
```

If you navigate to '/src/app/services/' and inspect 'oauth2.service.ts' you can take a look at the methods.

[...]

---

### 5. All methods provided <a name="5"></a>

All method usage examples were made in the 'app.component.ts' from the 'testbench' project. Just add a button to the 'app.component.html', which executes the methods.
```
// In our 'app.component.html'
<h3>AppComponent</h3>
<button (click)=onButtonClick()>Example</button>
<router-outlet>

// In our 'app.component.ts'
export class AppComponent {
  constructor(private service: OAuth2Service) { }

  onButtonClick() {
    // logic that gets executed when you press the button
  }
}
```
__All methods:__

1. ```'openLogin(): void'``` - redirects the user to the login form. 
```
onButtonClick() {
    this.service.openLogin();
}
```
2. ```'openRegister(): void'``` - redirects the user to the register form.
```
onButtonClick() {
    this.service.openRegister();
}
```
3. ```'openChangePassword(): void'``` - redirects the user to change his/her password
```
onButtonClick() {
    this.service.openChangePassword();
}
```
4. ```'requestToken(code: string): void'``` - requests an access token from the OAuth server. You have to pass an AuthCode as a parameter.
```
code: string = ""; // <-- You have to obtain the AuthCode from the URL first.
onButtonClick() {
    this.service.requesToken(this.code);
}
```
5. ```'refreshToken(): void'``` - takes your refresh token and requests a new access token.
```
onButtonClick() {
    this.service.refreshToken();
}
```
6. ```'logout(): void'``` - removes your current access token and refresh token from your localStorage and redirects you to your '/redirect' route.
```
onButtonClick() {
    this.service.logout();
}
```
7. ```'isAuthenticated(): boolean'``` - returns true, if the user has an access token in their localStorage.
```
onButtonClick() {
    if(this.service.isAuthenticated())
      console.log("success");
}
```
8. ```'getOauthScope(): string'``` - returns a string containing the scopes you specified in your config.
```
onButtonClick() {
    scopes = this.service.getOauthScope();
}
```
9. ```'getOauthClientSecret(): string'``` - returns the 'client secret' from the config.
```
onButtonClick() {
    secret = this.service.getOauthClientSecret();
}
```
10. ```'getOauthClientId(): string'``` - returns the 'client id' from the config.
```
onButtonClick() {
    id = this.service.getOauthClientId();
}
```
11. ```'getOauthHost(): string'``` - returns the 'oauthBaseURL' from the config.
```
onButtonClick() {
    baseUrl = this.service.getOauthHost();
}
```
12. ```'getAccessToken(): string'``` - returns the currently stored access token.
```
onButtonClick() {
    token = this.service.getAccessToken();
}
```
13. ```'getRefreshToken(): string'``` - returns the currently stored refresh token.
```
onButtonClick() {
    refreshToken = this.service.getRefreshToken();
}
```

> If you have any questions regarding the above methods you can take a look into the services code. ('oauth' project)

---

### 6. Support <a name="6"></a>
###### [(back to top)](#7)

If you have any questions regarding the usage of the library you can contact us at: [...]