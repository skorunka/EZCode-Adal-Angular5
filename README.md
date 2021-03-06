# EZCodeAdalAngular5

This library is a Azure Active Directory Authentication Library (adal.js) wrapper package for Angular 5. It can be used to authenticate Angular 5 application for Azure Active Directory and generate token to communicate to MS Graph API and 3rd party Web API secured by Azure AD. 

## Installation

Run the following command to install the package. 
`npm install ezcode-adal-angular5 --save`

## Authenticate Usage
1. Create a adal configuration file ezcodeadalconfig.ts under service folder
```typescript
import {IEZCodeAdalConfig} from 'ezcode-adal-angular5/lib/IEZCodeAdalConfig';

export const ezcodeAdalConfigLocal: IEZCodeAdalConfig={
    tenant: '[your tenant name/id]',
    clientId: '[your client id]',
    redirectUri: window.location.href.substring(0, window.location.href.lastIndexOf("/")+1), 
    postLogoutRedirectUri: window.location.origin + '/',
    endpoints: {
        'https://graph.microsoft.com/v1.0/me': 'https://graph.microsoft.com',
        '[webapi url]': '[webapi resource id]'

    }
};
```
2. Update app.module.ts to include the ezcode-adal-angular5 library. Make sure you set useHash to true because adal relies on hash to return the token. 
```typescript
import { EZCodeAdalService} from 'ezcode-adal-angular5/lib/ezcode-adal.service';

//import config for local.
import { ezcodeAdalConfig } from './services/ezcodeAdalConfig';



@NgModule({
    declarations: [
        ...
    ],
    imports: [
        ...
        EZCodeAdalModule.forRoot(ezcodeAdalConfigLocal),
        RouterModule.forRoot(rootRouterConfig, { useHash: true })
    ],
    providers: [
        EZCodeAdalService,
        ...
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```
3. Add EZCodeAdalComponentGuard to secure your component. When users click the secured components, application will redirect them to Azure AD login page. 
```typescript
// import { ValueComponent } from './values/value.controller';
import { Component } from '@angular/core';

import { Routes } from '@angular/router';

import { AboutComponent } from './about/about.component';
import { HomeComponent } from './home/home.component';
import { OrderComponent } from './order/order.component';
import {MsgraphComponent} from './msgraph/msgraph.component';

import {EZCodeAdalComponentGuard} from 'ezcode-adal-angular5/lib/ezcode-adal-component.guard';

export const rootRouterConfig: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent }, //, canActivate: [EZCodeAdalComponentGuard]
  { path: 'order', component: OrderComponent, canActivate: [EZCodeAdalComponentGuard]},
  { path: 'msgraph', component: MsgraphComponent, canActivate: [EZCodeAdalComponentGuard]},
  { path: 'about', component: AboutComponent }
];
```

## Usage for consuming a web api
1. If you need to call MS Graph API or 3rd party Web API, You need to add your MS Graph API endpoint and resource id to ezcodeadalconfig.ts
```typescript
import {IEZCodeAdalConfig} from 'ezcode-adal-angular5/lib/IEZCodeAdalConfig';

export const ezcodeAdalConfigLocal: IEZCodeAdalConfig={
    tenant: '[your tenant name/id]',
    clientId: '[your client id]',
    redirectUri: window.location.href.substring(0, window.location.href.lastIndexOf("/")+1), 
    postLogoutRedirectUri: window.location.origin + '/',
    endpoints: {
        'https://graph.microsoft.com/v1.0/me': 'https://graph.microsoft.com',
        '[webapi url]': '[webapi resource id]'

    }
};
```
2. Call a MS Graph API

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders,HttpResponse } from '@angular/common/http';

import { Observable } from 'rxjs/Rx';
import { of } from 'rxjs/observable/of';
import { catchError, map, tap } from 'rxjs/operators';
import 'rxjs/add/observable/throw';
import { IAllowance } from './IAllowance';
import { BaseService } from './base.service';
import { IJsonObject } from './IJsonObject';


@Injectable()
export class MsgraphService  {
    private _headers: HttpHeaders;
    constructor(httpClient: HttpClient) {
        this._headers = new HttpHeaders({ 'Content-Type': 'application/json' });
    }
    /**
     * getOrders
     */
    public getMe(): Observable<IJsonObject[]> {
        const url = "https://graph.microsoft.com/v1.0/me";

        //return the body json text. 
        return this.httpClient.get<any>(url, { headers: this._headers, observe: 'response' })
            .pipe(
                tap(result => this.log('fetched heroes')),
                catchError(this.handleError('getMe', [])),
                map((response:HttpResponse<any>)=>{
                    return this.getJsonObject(response.body);
                })
            );
            
    }

}
```
## Sample solution
you can find an sample solution from [ezcode-adal-angular5-sample](https://github.com/frankchen76/EZCode-Adal-Angular5-Sample) which was built based on Angular 5 and Bootstrap 4. 
The application itself was secured by a Azure AD App using implicit authentication flow. "MS Graph" route view is secured by `EZCodeAdalComponentGuard`. 
If an unauthenticated user accesses this view, the application will redirect to the Azure AD Login page. 
![alt text](https://public.dm.files.1drv.com/y4maciR4b07TLoQLlEQCi3doLxcDNsFJQbg2a0zGKWWRFJQvN0WWLRGMcVBJFkzLSRdikkCkjG1unDYsYk2cMhRe39-5U1vs8-U9M342TpxkX6vYQy2qZ-o5bgE-1hdH_6k1Mm-JFVxquu09bbYyYVgN_cmrH8sVXeWWWfRZe2uEdHmJlQWuKO1ukGH1Ytv9ZZZ0kIKgTnzn-0ZxKGaqFvtSg/ezcode-adal-angular5-sample.jpg?psid=1)

## Change log
* 1.0.7: the initial version was released.
* 1.0.8: Fixed a issue that Angular Http cannot make a non-auth API .
