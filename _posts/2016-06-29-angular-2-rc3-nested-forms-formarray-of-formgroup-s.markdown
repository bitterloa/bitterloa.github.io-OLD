---
published: true
title: Angular 2 (rc3) Nested Forms: FormArray of FormGroup(s)
layout: post
tags: [angular2, forms, FormGroup, FormArray]
---
Took me a while to figure out how to do something that was so simple in Angular 1; a form with dynamically added/removed form controls that syncs with both the model and form object. This outlines my solution with the latest version of Angular 2rc3 (latest at the time of this writing).

Here is a [working example of the code](http://plnkr.co/edit/nHSIsciSZNTQzQjxkXsk?p=preview).

Also see this document showing the [newest ways to use forms in Angular 2rc3](https://docs.google.com/document/u/1/d/1RIezQqE4aEhBRmArIAS1mRIZtWFf6JxN_7B4meyWK0Y/pub).

```javascript
import { Component } from '@angular/core';
import {REACTIVE_FORM_DIRECTIVES, FormControl, FormGroup, FormArray} from '@angular/forms';

@Component({
  selector: 'my-app',
  templateUrl: 'app/app.html',
  directives: [REACTIVE_FORM_DIRECTIVES],
  providers:  []
})
export class AppComponent {
  form: FormGroup;
  myModel:any;

  constructor() {
    // initializing a model for the form to keep in sync with. 
    // usually you'd grab this from a backend API
    this.myModel = {
      name: "Joanna Jedrzejczyk",
      payOffs: [
        {amount: 111.11, date: "Jan 1, 2016", final: false},
        {amount: 222.22, date: "Jan 2, 2016", final: true}
        ]
    }
    
    // initialize form with empty FormArray for payOffs
    this.form = new FormGroup({
      name: new FormControl(''),
      payOffs: new FormArray([])
    });

    // now we manually use the model and push a FormGroup into the form's FormArray for each PayOff
    this.myModel.payOffs.forEach( 
      (po) => 
        this.form.controls.payOffs.push(this.createPayOffFormGroup(po))
    );
  }

  createPayOffFormGroup(payOffObj) {
    console.log("payOffObj", payOffObj);
    return new FormGroup({
      amount: new FormControl(payOffObj.amount),
      date: new FormControl(payOffObj.date),
      final: new FormControl(payOffObj.final)
    });
  }
  
  addPayOff(event) {
    event.preventDefault(); // ensure this button doesn't try to submit the form
    var emptyPayOff = {amount: null, date: null, final: false};
    
    // add pay off to both the model and to form controls because I don't think Angular has any way to do this automagically yet
    this.myModel.payOffs.push(emptyPayOff);
    this.form.controls.payOffs.push(this.createPayOffFormGroup(emptyPayOff));
    console.log("Added New Pay Off", this.form.controls.payOffs)
  }
  
  deletePayOff(index:number) {
    // delete payoff from both the model and the FormArray
    this.myModel.payOffs.splice(index, 1);
    this.form.controls.payOffs.removeAt(index);
  }
}
```