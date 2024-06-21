# Ass2

## Account Page

```ts

// Import required libraries and components
import { Component, ViewChild, TemplateRef } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { ModalController } from '@ionic/angular';
import { IonModal } from '@ionic/angular';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { IonicModule } from '@ionic/angular';
import { ReactiveFormsModule } from '@angular/forms';

// Define the component metadata
@Component({
  selector: 'app-account',
  templateUrl: './account-page.page.html',
  styleUrls: ['./account-page.page.scss'],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule, ReactiveFormsModule],
})
export class AccountPage {
  user = {
    name: 'John Doe',
    email: 'john.doe@example.com',
    phone: '123-456-7890',
  };
  pastTotal: number = 0;
  @ViewChild('getHelpModal', { static: true }) getHelpModal!: TemplateRef<any>;

  @ViewChild(IonModal) modal!: IonModal;
  pastOrders: any[] = [];

  // Lifecycle hook that runs before the view enters
  ionViewWillEnter() {
    this.loadPastOrders();
  }

  // Load past orders from localStorage
  loadPastOrders() {
    const paymentList = JSON.parse(localStorage.getItem('paymentList') ?? '[]');
    this.pastOrders = paymentList;
    this.pastTotal = this.getTotalPrice(); // Calculate the pastTotal using the getTotalPrice() function
  }

  // Calculate the total price of past orders
  getTotalPrice() {
    let sum = 0;
    for (let order of this.pastOrders) {
      sum += order.price * order.quantity;
    }
    return sum;
  }

  editUserDetailsForm: FormGroup;

  constructor(
    private modalController: ModalController,
    private formBuilder: FormBuilder
  ) {
    this.editUserDetailsForm = this.formBuilder.group({
      name: [this.user.name, Validators.required],
      email: [this.user.email, [Validators.required, Validators.email]],
      phone: [this.user.phone, Validators.required],
    });
  }

  // Dismiss the modal
  dismissModal() {
    this.modal.dismiss(null, 'cancel');
  }

  // Close the modal
  closeModal() {
    this.modalController.dismiss(null, 'cancel');
  }

  // Update user details based on the form data
  updateUserDetails() {
    if (this.editUserDetailsForm.valid) {
      this.user.name = this.editUserDetailsForm.value.name;
      this.user.email = this.editUserDetailsForm.value.email;
      this.user.phone = this.editUserDetailsForm.value.phone;
    }
  }

  // Reorder the past orders
  reorder() {
    // Get the contents of the paymentList local storage
    const paymentList = JSON.parse(localStorage.getItem('paymentList') ?? '[]');
    const orders = paymentList;

    // Save the past orders to local storage
    localStorage.setItem('cart', JSON.stringify(orders));

    // Clear the paymentList after reordering
    localStorage.setItem('paymentList', JSON.stringify([]));

    // Navigate back to the cart page
    window.history.back();
  }
}
```

## Accound Page HTML

```html

<ion-header>
  <ion-toolbar>
    <ion-title>Account</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>
  <ion-card>
    <ion-card-header>
      <ion-grid>
        <ion-row align-items-center justify-content-start>
          <ion-col size="auto">
            <ion-icon name="person-outline" class="big-icon"></ion-icon>
          </ion-col>
          <ion-col size="auto">
            <ion-card-title>User Details</ion-card-title>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-card-header>

    <ion-card-content>
      <p>Name: <span>{{ user.name }}</span></p>
      <p>Email: <span>{{ user.email }}</span></p>
      <p>Phone: <span>{{ user.phone }}</span></p>
      <ion-button id="open-edit-user-details-modal" expand="block"
        >Edit User Details</ion-button
      >
      <ion-modal trigger="open-edit-user-details-modal">
        <ng-template #editUserDetailsModal>
          <ion-header>
            <ion-toolbar>
              <ion-title>Edit User Details</ion-title>
            </ion-toolbar>
          </ion-header>
          <ion-content>
            <form
              [formGroup]="editUserDetailsForm"
              (ngSubmit)="updateUserDetails()"
            >
              <ion-item>
                <ion-label>Name:</ion-label>
                <ion-input formControlName="name"></ion-input>
              </ion-item>
              <ion-item>
                <ion-label>Email:</ion-label>
                <ion-input formControlName="email"></ion-input>
              </ion-item>
              <ion-item>
                <ion-label>Phone:</ion-label>
                <ion-input formControlName="phone"></ion-input>
              </ion-item>
              <ion-button expand="block" type="submit">Save</ion-button>
              <ion-button expand="block" (click)="dismissModal()"
                >Close</ion-button
              >
            </form>
          </ion-content>
        </ng-template>
      </ion-modal>
    </ion-card-content>
  </ion-card>

  <ion-card>
    <ion-card-content>
      <ion-grid>
        <ion-row>
          <ion-col size="auto" class="ion-align-self-center">
            <ion-icon name="home-outline" class="big-icon"></ion-icon>
          </ion-col>
          <ion-col class="ion-align-self-center">
            <ion-card-title>Manage Addresses</ion-card-title>
          </ion-col>
          <ion-col size="auto" text-right class="ion-align-self-center">
            <ion-button fill="clear" size="small">
              <ion-icon
                slot="icon-only"
                name="chevron-forward-outline"
              ></ion-icon>
            </ion-button>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-card-content>
  </ion-card>

  <ion-card>
    <ion-card-header>
      <ion-grid>
        <ion-row align-items-center justify-content-start>
          <ion-col size="auto">
            <ion-icon name="clipboard-outline" class="big-icon"></ion-icon>
          </ion-col>
          <ion-col size="auto">
            <ion-card-title>Past Orders</ion-card-title>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-card-header>
    <ion-card-content>
      <ion-list>
        <ion-item *ngFor="let order of pastOrders">
          <ion-col size="12">
            <ion-card>
              <ion-card-content>
                <ion-item lines="none">
                  <ion-thumbnail slot="start">
                    <img alt="Restaurant image" [src]="order.restaurantImage" />
                  </ion-thumbnail>
                  <ion-label>
                    <h2>{{ order.restaurantName }}</h2>
                    <p>{{ order.price | currency:'ZAR':'symbol' }}</p>
                  </ion-label>
                </ion-item>
                <br />
                <ion-badge>Quantity: {{ order.quantity }}</ion-badge>
              </ion-card-content>
            </ion-card>
          </ion-col>
        </ion-item>
      </ion-list>
      <br />
      <h2>Total: {{ pastTotal | currency:'ZAR':'symbol' }}</h2>
      <br />
      <ion-button expand="block" (click)="reorder()">Reorder</ion-button>
    </ion-card-content>
  </ion-card>

  <br />

  <ion-row>
    <ion-col>
      <ion-button fill="solid" expand="block" id="open-get-help-modal">
        <ion-icon name="help-circle"></ion-icon>
        Get Help
      </ion-button>
    </ion-col>
  </ion-row>

  <ion-modal trigger="open-get-help-modal">
    <ng-template #getHelpModal>
      <ion-header>
        <ion-toolbar>
          <ion-icon name="help-circle"></ion-icon>
          <ion-title>Get Help</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content>
        <ion-card>
          <ion-card-header>
            <ion-card-title> Getting Started </ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p>
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec
              eleifend nisl sit amet lectus laoreet hendrerit.
            </p>
            <p>
              Sed ornare nisi a arcu blandit, quis mollis purus lobortis.
              Pellentesque habitant morbi tristique senectus et netus et
              malesuada fames ac turpis egestas.
            </p>
          </ion-card-content>
        </ion-card>

        <ion-card>
          <ion-card-header>
            <ion-card-title> Troubleshooting </ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p>
              Sed vulputate, nisi at aliquet posuere, justo diam suscipit lorem,
              sit amet rhoncus nunc ex nec nulla.
            </p>
            <p>
              Suspendisse vel dolor auctor, blandit sapien eu, fringilla nibh.
              Integer gravida ante eget felis volutpat ullamcorper.
            </p>
          </ion-card-content>
        </ion-card>

        <ion-card>
          <ion-card-header>
            <ion-card-title> Contact Us </ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p>
              If you have any questions or need further assistance, please feel
              free to contact us at:
            </p>
            <p>Email: support@example.com</p>
            <p>Phone: 1-800-555-1234</p>
          </ion-card-content>
        </ion-card>
        <ion-button expand="block" (click)="closeModal()">Close</ion-button>
      </ion-content>
    </ng-template>
  </ion-modal>
</ion-content>
```

## Cart Page

```ts

// Import required libraries and components
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { IonicModule } from '@ionic/angular';
import { Component, ViewChild } from '@angular/core';
import { IonModal } from '@ionic/angular';

// Define an interface for cart items
interface CartItem {
  restaurantName: string;
  slogan: string;
  dishPrice: number;
  quantity: number;
  restaurantImage: string;
}

// Define the component metadata
@Component({
  selector: 'app-cart-page',
  templateUrl: './cart-page.page.html',
  styleUrls: ['./cart-page.page.scss'],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule],
})
export class CartPagePage {
  cart: any[] = [];
  pastOrders: any[] = [];
  deliveryFee: number = 5.0;
  deliveryInstructions: string = '';
  paymentMessage: string = 'Payment was successful';

  // Get a reference to the paymentSuccessModal element
  @ViewChild('paymentSuccessModal', { static: false })
  paymentSuccessModal!: IonModal;

  constructor() {}

  // Load the current order list from localStorage
  loadOrders() {
    this.cart = JSON.parse(localStorage.getItem('cart') ?? '[]');
  }

  // Load past orders from localStorage
  loadPastOrders() {
    this.pastOrders = JSON.parse(localStorage.getItem('paymentList') ?? '[]');
  }

  // Lifecycle hook that runs before the view enters
  ionViewWillEnter() {
    this.loadOrders();
    this.loadPastOrders();
  }

  // Calculate the total cost of items in the cart
  total() {
    let sum = 0;
    for (let item of this.cart) {
      sum += item.price * item.quantity;
    }
    return sum + this.deliveryFee;
  }

  // Make a payment and update localStorage
  makePayment() {
    this.openModal(); // Open the payment success modal
    const newPaymentList = this.cart;
    localStorage.setItem('paymentList', JSON.stringify(newPaymentList));
    localStorage.setItem('cart', JSON.stringify([])); // Clear the orderList in localStorage
  }

  // Clear the cart and remove orderList from localStorage
  clearCart() {
    this.cart = [];
    localStorage.removeItem('cart');
  }

  // Remove an item from the cart and update localStorage
  removeItem(item: CartItem) {
    const index = this.cart.indexOf(item);
    if (index > -1) {
      this.cart.splice(index, 1);
      localStorage.setItem('cart', JSON.stringify(this.cart));
    }
  }

  // Open the payment success modal
  openModal() {
    this.paymentSuccessModal.present();
  }

  // Close the payment success modal
  closeModal(modal: IonModal) {
    modal.dismiss();
  }
}
```

## Cart Page HTML

```html

<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>Cart</ion-title>
    <ion-buttons slot="end">
      <ion-button (click)="clearCart()">
        <ion-icon slot="icon-only" name="trash"></ion-icon>
      </ion-button>
      <ion-label>Clear Cart</ion-label>
    </ion-buttons>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-card *ngFor="let item of cart">
    <ion-card-content>
      <ion-item lines="none">
        <ion-buttons slot="end">
          <ion-button (click)="removeItem(item)">
            <ion-icon slot="icon-only" name="trash"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-thumbnail slot="start">
          <img alt="Restaurant image" [src]="item.restaurantImage" />
          <!--IMAGE-->
        </ion-thumbnail>
        <ion-label>
          <h3>{{ item.restaurantName }}</h3>
          <p>{{ item.slogan }}</p>
          <p>Price: {{ item.price | currency:'ZAR':'symbol' }}</p>
        </ion-label>
      </ion-item>
      <ion-item>
        <ion-label>Quantity:</ion-label>
        <ion-input
          [(ngModel)]="item.quantity"
          type="number"
          min="1"
        ></ion-input>
      </ion-item>
    </ion-card-content>
  </ion-card>

  <ion-item>
    <ion-label>Delivery Fee:</ion-label>
    <ion-text>{{ deliveryFee | currency:'ZAR':'symbol' }}</ion-text>
  </ion-item>

  <ion-item>
    <ion-label>Total:</ion-label>
    <ion-text>{{ total() | currency:'ZAR':'symbol' }}</ion-text>
  </ion-item>

  <ion-item>
    <ion-textarea
      placeholder="Delivery Instructions"
      [(ngModel)]="deliveryInstructions"
    ></ion-textarea>
  </ion-item>

  <ion-button expand="full" id="make-payment" (click)="openModal()"
    >Make Payment</ion-button
  >
  <!-- <ion-toast trigger="make-payment" [message]="paymentMessage" [duration]="5000"></ion-toast> -->

  <ion-modal id="payment-success-modal" #paymentSuccessModal>
    <ng-template>
      <div class="wrapper">
        <h1>Payment Success</h1>
        <ion-button
          expand="full"
          (click)="closeModal(paymentSuccessModal); makePayment()"
          id="complete-button"
          >Complete</ion-button
        >
      </div>
    </ng-template>
  </ion-modal>
</ion-content>
```

## Home Page

```ts

// Import required libraries and components
import { Component } from '@angular/core';
import { IonicModule } from '@ionic/angular';
import { CommonModule } from '@angular/common';
// import { IonContent, IonPage, IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle, IonCardContent, IonIcon, IonButton } from '@ionic/vue';

// Define the component metadata
@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
  standalone: true,
  imports: [IonicModule, CommonModule],
})
export class HomePage {
  // Initialize a list of restaurants

  restaurantList = [
    {
      name: 'KFC',
      slogan: "Well, it's finger lickin' good",
      rating: 4.3,
      distance: '5km',
      priceRange: 48.93,
      image: 'assets/Pictures/KFCLOGO.svg.png',
    },
    {
      name: 'Steers',
      slogan: 'Real food. Made Real Good',
      rating: 4.8,
      distance: '3km',
      priceRange: 54.9,
      image: '/assets/Pictures/STEERSLOGO.png',
    },
    {
      name: 'Debonairs',
      slogan: 'WHO WE ARE IS AMAZING.',
      rating: 3.8,
      distance: '2,1km',
      priceRange: 78.92,
      image: '/assets/Pictures/DEBONAIRSLOGO.png',
    },
    {
      name: 'Wimpy',
      slogan: 'We Love It When You Talk Local',
      rating: 4.1,
      distance: '1.9km',
      priceRange: 63.43,
      image: '/assets/Pictures/WIMPYLOGO.svg.png',
    },
  ];

  // Constructor for the HomePage class
  constructor() {}

  // Dismiss the modal if it is open
  dismissModal() {
    const modalElement = document.querySelector('ion-modal');
    if (modalElement) {
      modalElement.dismiss();
    }
  }

  // Add a featured restaurant to the cart
  addFeaturedToCart() {
    // Define the featured restaurant
    const featuredRestaurant = {
      name: 'Steers',
      slogan: 'Real food. Made real good.',
      rating: 4.5,
      distance: '1.2 km',
      priceRange: 54.9,
      image: '/assets/Pictures/STEERSLOGO.png',
    };

    // Call addToCart method with the featured restaurant as an argument
    this.addToCart(featuredRestaurant);
  }

  addToCart(restaurant: {
    name: string;
    slogan: string;
    rating: number;
    distance: string;
    priceRange: number;
    image: string;
  }) {
    // Retrieve the cart from local storage, or create an empty array if it doesn't exist
    const cart = JSON.parse(localStorage.getItem('cart') ?? '[]');

    // Find the index of the restaurant in the cart, if it exists
    const itemIndex = cart.findIndex(
      (order: { restaurantName: string }) =>
        order.restaurantName === restaurant.name
    );

    // If the restaurant is not in the cart, add it with a quantity of 1
    if (itemIndex === -1) {
      cart.push({
        price: restaurant.priceRange,
        restaurantName: restaurant.name,
        restaurantImage: restaurant.image,
        quantity: 1,
      });
    } else {
      // If the restaurant is already in the cart, increment its quantity
      cart[itemIndex].quantity++;
    }
    // Save the updated cart to local storage
    localStorage.setItem('cart', JSON.stringify(cart));
  }
}
```

## Home Page HTML

```html

<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title> Home </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true" class="ios-custom-content">
  <ion-header collapse="condense">
    <ion-toolbar>
      
    </ion-toolbar>
  </ion-header>
  <img src="/assets/Pictures/Nandos food.png">
  <ion-grid>
      <ion-row>
      <ion-col size="12">
        <h2 class="ion-text-center ion-padding">All Restaurants</h2>
      </ion-col>
    </ion-row>

    <ion-grid>
      <ion-row>
        <ion-col size="12" *ngFor="let restaurant of restaurantList">
          <ion-card>
            <ion-card-content>
              <ion-item lines="none">
                <ion-thumbnail slot="start">
                  <img alt="Restaurant image" [src]="restaurant.image" />
                  <!--IMAGE-->
                </ion-thumbnail>
                <ion-label>
                  <h3>{{ restaurant.name }}</h3>
                  <!--NAME-->
                  <p>{{ restaurant.slogan }}</p>
                  <!--SLOGAN-->
                  <p style="color: red">
                    Price: {{ restaurant.priceRange | currency:'ZAR':'symbol' }}
                  </p>
                  <!--PRICE RANGE-->
                </ion-label>
              </ion-item>
              <div
                style="
                  display: flex;
                  justify-content: space-between;
                  align-items: center;
                "
              >
                <div style="display: flex; align-items: center">
                  <ion-icon
                    name="star"
                    style="margin-right: 5px; color: gold"
                  ></ion-icon>
                  <span style="color: gold">{{ restaurant.rating }}</span>
                  <!--RATING-->
                </div>
                <div style="display: flex; align-items: center">
                  <span>{{ restaurant.distance }}</span>
                  <!--DISTANCE-->
                </div>
                <ion-button
                  (click)="addToCart(restaurant);"
                  size="small"
                  color="primary"
                  style="align-self: flex-start"
                >
                  <ion-icon slot="icon-only" name="cart"></ion-icon>
                  Order Now
                </ion-button>
              </div>
            </ion-card-content>
          </ion-card>
        </ion-col>
      </ion-row>
    </ion-grid>
  </ion-grid></ion-content
>
```

## Search Page

```ts

// Import required libraries and components
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { IonicModule } from '@ionic/angular';
import { HomePage } from '../home/home.page';

// Define the component metadata
@Component({
  selector: 'app-search-page',
  templateUrl: './search-page.page.html',
  styleUrls: ['./search-page.page.scss'],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule],
  providers: [HomePage], // Provide HomePage as a dependency to use its methods
})
export class SearchPagePage implements OnInit {
  searchQuery = ''; // Initialize the search query
  searchResults: any[] = []; // Initialize an array to store search results

  // Constructor for the SearchPagePage class
  constructor(private homePage: HomePage) {}

  // ngOnInit lifecycle hook, which is called when the component is initialized
  ngOnInit() {}

  // Perform the search for restaurants based on the search query
  performSearch() {
    // If the search query is not empty
    if (this.searchQuery.length > 0) {
      // Filter the restaurant list from HomePage using the search query
      this.searchResults = this.homePage['restaurantList'].filter(
        (restaurant: any) =>
          restaurant.name
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase()) ||
          restaurant.rating.toString().includes(this.searchQuery)
      );
    } else {
      // If the search query is empty, clear the search results
      this.searchResults = [];
    }
  }

  // Add a restaurant to the cart by calling the addToCart method from HomePage
  addToCart(restaurant: any) {
    this.homePage['addToCart'](restaurant);
  }
}
```

## Search Page HTML

```html

<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-buttons slot="start">
      <ion-back-button defaultHref="/"></ion-back-button>
    </ion-buttons>
    <ion-title>INF Delivery</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-searchbar
    placeholder="Search for restaurants or food"
    [(ngModel)]="searchQuery"
    (ionInput)="performSearch()"
  ></ion-searchbar>

  <ion-grid *ngIf="searchResults.length > 0">
    <ion-row>
      <ion-col size="12">
        <ion-label>Search Results</ion-label>
      </ion-col>
    </ion-row>
    <ion-row>
      <ion-col
        size="12"
        *ngFor="let result of searchResults"
        (click)="addToCart(result);"
      >
        <ion-card>
          <ion-card-content>
            <ion-item lines="none">
              <ion-thumbnail slot="start">
                <img [src]="result.image" />
              </ion-thumbnail>
              <ion-label>
                <h3>{{ result.name }}</h3>
                <p>{{ result.slogan }}</p>
                <p>{{ result.priceRange }}</p>
              </ion-label>
            </ion-item>
            <div
              style="
                display: flex;
                justify-content: space-between;
                align-items: center;
              "
            >
              <div style="display: flex; align-items: center">
                <ion-icon name="star" style="margin-right: 5px"></ion-icon>
                <span>{{ result.rating }}</span>
              </div>
              <div style="display: flex; align-items: center">
                <span>{{ result.distance }}</span>
              </div>
              <ion-button
                size="small"
                color="primary"
                style="align-self: flex-start"
              >
                <ion-icon slot="icon-only" name="cart"></ion-icon>
                Order Now
              </ion-button>
            </div>
          </ion-card-content>
        </ion-card>
      </ion-col>
    </ion-row>
  </ion-grid>
</ion-content>
```

## Tabs Page

```ts

import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { IonicModule } from '@ionic/angular';

@Component({
  selector: 'app-tabs',
  templateUrl: './tabs.page.html',
  styleUrls: ['./tabs.page.scss'],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule],
})
export class TabsPage implements OnInit {
  constructor() {}

  ngOnInit() {}
}
```

## Tabs Page HTML

```html

<ion-tabs>
  <ion-tab-bar
    slot="bottom"
    style="--ion-safe-area-bottom: env(safe-area-inset-bottom)"
  >
    <ion-tab-button tab="home">
      <ion-icon aria-hidden="true" name="home"></ion-icon>
      <ion-label>Home</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="search">
      <ion-icon aria-hidden="true" name="search"></ion-icon>
      <ion-label>Search</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="cart">
      <ion-icon aria-hidden="true" name="cart"></ion-icon>
      <ion-label>Cart</ion-label>
    </ion-tab-button>

    <ion-tab-button tab="account">
      <ion-icon aria-hidden="true" name="person"></ion-icon>
      <ion-label>Account</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

## Tabs Route

```ts

// Import required libraries and components
import { Routes } from '@angular/router';
import { TabsPage } from './tabs.page';

// Define the routes for the application
export const routes: Routes = [
  {
    path: 'tabs', // Route for the tabs component
    component: TabsPage,
    children: [
      // Define child routes for each tab
      {
        path: 'home', // Route for the home tab
        loadComponent: () =>
          // Lazy load the HomePage component when the route is accessed
          import('../home/home.page').then((m) => m.HomePage),
      },
      {
        path: '', // Default route when no specific path is given
        redirectTo: '/tabs/home', // Redirect to the home tab
        pathMatch: 'full',
      },
      {
        path: 'search', // Route for the search tab
        loadComponent: () =>
          // Lazy load the SearchPagePage component when the route is accessed
          import('../search-page/search-page.page').then(
            (m) => m.SearchPagePage
          ),
      },
      {
        path: 'cart', // Route for the cart tab
        loadComponent: () =>
          // Lazy load the CartPagePage component when the route is accessed
          import('../cart-page/cart-page.page').then((m) => m.CartPagePage),
      },
      {
        path: 'account', // Route for the account tab
        loadComponent: () =>
          // Lazy load the AccountPage component when the route is accessed
          import('../account-page/account-page.page').then(
            (m) => m.AccountPage
          ),
      },
    ],
  },
  {
    path: '', // Default route when no specific path is given
    redirectTo: '/tabs/home', // Redirect to the home tab
    pathMatch: 'full',
  },
];
```
