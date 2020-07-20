# Angular 9 Signature Pad

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 9.0.2.

# Create Project
`ng new A9SignaturePad`

# Install Signature Pad
`npm install --save signature_pad`

# Import Signature Pad
`import SignaturePad from 'signature_pad';`

# Use In Code
` this.signaturePad = new SignaturePad(this.signaturePadElement.nativeElement);`

# Template Code
`<canvas #sPad width="900" height="600" style="touch-action: none;"></canvas>`

## Example
### HTML
```html
<ion-header>
  <ion-toolbar color="success">
    <ion-buttons slot="end">
      <ion-button color="light" expand="full" (click)="modal.dismiss()"> Close</ion-button>
    </ion-buttons>
    <ion-title>Signature</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
  <div class="signature-pad">
    <div>
      <canvas class="signature-pad-canvas" #sPad width="900" height="600" style="touch-action: none;"></canvas>
    </div>
  </div>

  <ion-toolbar>
    <ion-button (click)="clear()">
      <ion-icon name="trash"> </ion-icon> CLEAR
    </ion-button>
    <ion-button (click)="undo()">
      <ion-icon name="arrow-undo"> </ion-icon> UNDO
    </ion-button>
    <ion-button color="success" [disabled]="isCanvasBlank()" (click)="save()">
      <ion-icon name="checkmark"> </ion-icon> SAVE
    </ion-button>
  </ion-toolbar>
</ion-content>
```

### CSS
```css
canvas {
  border: 3px dashed gray;
  width: 100%;
}
```

### TS
```typescript
import { Component, OnInit, ViewChild, HostListener, ElementRef, AfterViewInit } from '@angular/core';
import { ModalController } from '@ionic/angular';
import { FormGroup } from '@angular/forms';
import SignaturePad from 'signature_pad';

@Component({
  selector: 'app-signature',
  templateUrl: './signature.page.html',
  styleUrls: ['./signature.page.scss'],
})
export class SignaturePage implements OnInit, AfterViewInit {

  @ViewChild('sPad', { static: true }) signaturePadElement;
  signaturePad: any;

  form: FormGroup;
  modal: any;
  signaturePadOptions: Object = {};
  public _signature: any = null;
  innerWidth: number;

  constructor(public modalCtrl: ModalController, private elementRef: ElementRef) { }

  /**
   * Sets the styles and initiates the canvas
   *
   * @memberof SignaturePage
   */
  ngOnInit(): void {
    this.innerWidth = window.innerWidth - 32;

    this.resize();

  }

  changeColor() {
    const r = Math.round(Math.random() * 255);
    const g = Math.round(Math.random() * 255);
    const b = Math.round(Math.random() * 255);
    const color = 'rgb(' + r + ',' + g + ',' + b + ')';
    this.signaturePad.penColor = color;
  }

  clear() {
    this.signaturePad.clear();
  }

  undo() {
    const data = this.signaturePad.toData();
    if (data) {
      data.pop(); // remove the last dot or line
      this.signaturePad.fromData(data);
    }
  }

  // FORCE to recreate signature pad
  @HostListener('window:resize', ['$event'])
  onResize(event) {
    this.resize();
  }

  resize() {
    const pagePadding = 32;
    const pageWidth = window.innerWidth;
    const pageheight = window.innerWidth / 2;
    this.innerWidth = pageWidth - pagePadding;

    const canvas: any = this.elementRef.nativeElement.querySelector('canvas');
    // When zoomed out to less than 100%, for some very strange reason,
    // some browsers report devicePixelRatio as less than 1
    // and only part of the canvas is cleared then.
    const ratio = Math.max(window.devicePixelRatio || 1, 1);

    // const canvas: any = this.signaturePad._canvas;
    canvas.width = innerWidth - pagePadding;
    canvas.height = pageheight;
    if (this.signaturePad) {
      this.signaturePad.clear(); // otherwise isEmpty() might return incorrect value

    }
  }

  public ngAfterViewInit(): void {
    this.signaturePad = new SignaturePad(this.signaturePadElement.nativeElement);

    this.signaturePad.clear();
  }

  /**
   * Closes the modal and passes the signature to the page where it is being requested
   *
   * @memberof SignaturePage
   */
  save(): void {
    const img = this.signaturePad.toDataURL('image/png');
    this.modal.dismiss({
      signatureImage: img,
      imageArray: this.signaturePad.toData()
    });
  }

  /**
   * returns true there is no signature, false if there is a signature
   *
   * @returns
   * @memberof SignaturePage
   */
  isCanvasBlank(): boolean {
    if (this.signaturePad) {
      return this.signaturePad.isEmpty() ? true : false;
    }
  }
}

```
