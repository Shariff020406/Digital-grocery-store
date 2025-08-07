
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/shopsmart');

const User = mongoose.model('User', {
  name: String,
  email: String,
  password: String,
  role: String // 'customer', 'seller', or 'admin'
});

const Product = mongoose.model('Product', {
  name: String,
  category: String,
  price: Number,
  description: String,
  image: String,
  stock: Number,
  sellerId: String
});

const Order = mongoose.model('Order', {
  userId: String,
  items: Array,
  total: Number,
  status: String // 'pending', 'shipped', etc.
});

// Routes
app.post('/register', async (req, res) => {
  const user = new User(req.body);
  await user.save();
  res.send(user);
});

app.post('/login', async (req, res) => {
  const user = await User.findOne({ email: req.body.email, password: req.body.password });
  if (user) res.send(user);
  else res.status(401).send('Invalid credentials');
});

app.post('/products', async (req, res) => {
  const product = new Product(req.body);
  await product.save();
  res.send(product);
});

app.get('/products', async (req, res) => {
  const products = await Product.find();
  res.send(products);
});

app.post('/orders', async (req, res) => {
  const order = new Order(req.body);
  await order.save();
  res.send(order);
});

app.get('/orders/:userId', async (req, res) => {
  const orders = await Order.find({ userId: req.params.userId });
  res.send(orders);
});

app.listen(3000, () => console.log('Server running on port 3000'));


// --- Frontend (Angular - Simplified Structure) ---
// Note: This is just a sample component file, Angular project setup assumed

// src/app/app.component.html
<!--
<div class="container">
  <h1>ShopSmart</h1>
  <div *ngFor="let product of products">
    <div class="card">
      <img [src]="product.image" class="card-img-top">
      <div class="card-body">
        <h5 class="card-title">{{product.name}}</h5>
        <p class="card-text">{{product.description}}</p>
        <p>₹{{product.price}}</p>
        <button (click)="addToCart(product)" class="btn btn-primary">Add to Cart</button>
      </div>
    </div>
  </div>
</div>
-->

// src/app/app.component.ts
import { HttpClient } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent implements OnInit {
  products: any[] = [];
  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.http.get<any[]>('http://localhost:3000/products').subscribe(data => {
      this.products = data;
    });
  }

  addToCart(product: any) {
    console.log('Added to cart:', product);
  }
}

