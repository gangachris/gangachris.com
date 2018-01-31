---
title: "Quicker Angular Forms Validation"
date: 2017-01-01T16:53:41+03:00
draft: false
---

The community is moving away from calling Angular by version names, Angular 2, Angular 4 and the likes. It’s now just Angular, mainly because of the clear 6 months release cycles recently mentioned.

So in light of this, I updated my Open Source Project, angular validators, to ng-validators. I was surprised that name was available, because there are a couple of validators out there, you can pick whichever you want.

[ng-validators](https://github.com/gangachris/ng-validators) is a wrapper around the widely known [validator js](https://github.com/chriso/validator.js) library, and since I didn’t feel like rewriting validation that already existed out there, I did this.

As of now, it only supports ReactiveFormsModule. FormsModule directives will be written some day, by me, or another open sourcerer.

Using it is quite simple.

Install it in your angular project with either npm or yarn
```
npm install --save ng-validators
```
or
```
yarn add ng-validators
```
Once done, make sure you have the ReactiveFormsModule included in your `app.modules.ts` file.

{{< highlight typescript >}}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';
import { AppComponent } from './app.component';
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    HttpModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

{{< / highlight >}}

Then in the component with the Form that you want to add validation to, add the FormBuilder, and the FormGroup providers. While, at it, add the ng-validators too.

{{< highlight typescript >}}
import { Component } from '@angular/core';
import { FormGroup, FormBuilder } from '@angular/forms';
import { NGValidators } from 'ng-validators';
Then now, create a Model Based form, as you would normally.

import { Component } from '@angular/core';
import { FormGroup, FormBuilder } from '@angular/forms';
import { NGValidators } from 'ng-validators';
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  theForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.theForm = fb.group({
      contains: ['', [NGValidators.contains('validate')]],
      equals: ['', [NGValidators.equals('validate')]],
      after: ['', [NGValidators.isAfter()]],
      alpha: ['', [NGValidators.isAlpha()]],
      email: ['', [NGValidators.isEmail()]]
     })
  }
}
{{< / highlight >}}
Templating for this should be intuitive if you’ve done a little of angular forms. I added a bootstrap CDN in my index file, then used the classes to update write the template below.

{{< highlight html >}}
<div class="container" [style.margin-top.px]="30">
  <form [formGroup]="theForm" novalidate>
      <div class="col-md-4">
        <div class="form-group">
          <label class="form-control-label">Contains</label>
          <input type="text" class="form-control form-control-success" name="contains" formControlName="contains">
          <div class="form-control-feedback">
            <div class="alert alert-danger" *ngIf="theForm.controls.contains.errors?.contains">Must contain 'validate'</div>
          </div>
        </div>
      </div>
  </form>
</div>
{{< / highlight >}}
So, I have all the validators provided by [validator js](https://github.com/chriso/validator.js), wrapped in this simple module.
