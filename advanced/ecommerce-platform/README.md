# Complete E-commerce Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1556742049-0cfed4f6a45d?w=600&q=80" alt="E-commerce Platform" width="600">
</p>

**Level:** Advanced  
**Estimated Time:** 4-8 weeks  
**Prerequisites:** Full-stack development, payment gateway integration, database design, security

---

## Description

Build a complete e-commerce platform with product catalog, shopping cart, secure payment processing, order management, and admin dashboard. This comprehensive project teaches you about transaction security, payment gateway integration, inventory management, order processing workflows, and building scalable online store systems.

The platform will handle product browsing, cart management, checkout flows, payment processing with Stripe or PayPal, order tracking, user authentication, product reviews, and a full admin panel for store management.

---

## Core Features

### Must Have
- User authentication (register, login, password reset)
- Product catalog with categories and search
- Product detail pages with images and descriptions
- Shopping cart (add, update, remove items)
- Checkout process with address management
- Payment gateway integration (Stripe/PayPal)
- Order confirmation and tracking
- User order history
- Admin dashboard for managing:
  - Products (CRUD operations)
  - Orders (view, update status)
  - Inventory
  - Basic analytics

### Nice to Have
- Product reviews and ratings
- Wishlist functionality
- Product recommendations
- Discount coupons and promotions
- Multiple payment methods
- Guest checkout option
- Email notifications (order confirmation, shipping)
- Product image zoom and gallery
- Stock management with low-stock alerts
- Advanced search with filters
- Multiple currencies support
- Shipping cost calculation
- Sales analytics and reports
- Customer support chat

---

## Learning Objectives

By building this project, you will learn:

- **E-commerce Architecture**: Designing scalable online store systems
- **Payment Integration**: Implementing secure payment gateways (Stripe, PayPal)
- **Transaction Security**: PCI compliance and secure checkout
- **Inventory Management**: Stock tracking and updates
- **Order Processing**: Complete order lifecycle management
- **Shopping Cart Logic**: Session management and cart persistence
- **Database Design**: Complex relationships (users, products, orders, reviews)
- **Image Handling**: Product image upload and optimization
- **Email Services**: Transactional emails (order confirmations, receipts)
- **Admin Interfaces**: Building management dashboards
- **SEO Optimization**: Product pages for search engines
- **Webhooks**: Handling payment status callbacks

---

## Implementation Guide

### Step 1: Set Up Project Structure

Create a complete full-stack e-commerce structure:

```
ecommerce-platform/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   ├── database.js
│   │   │   ├── stripe.js
│   │   │   └── email.js
│   │   ├── models/
│   │   │   ├── User.js
│   │   │   ├── Product.js
│   │   │   ├── Category.js
│   │   │   ├── Order.js
│   │   │   ├── Cart.js
│   │   │   └── Review.js
│   │   ├── controllers/
│   │   │   ├── authController.js
│   │   │   ├── productController.js
│   │   │   ├── cartController.js
│   │   │   ├── orderController.js
│   │   │   └── adminController.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── products.js
│   │   │   ├── cart.js
│   │   │   ├── orders.js
│   │   │   ├── payment.js
│   │   │   └── admin.js
│   │   ├── middleware/
│   │   │   ├── auth.js
│   │   │   ├── admin.js
│   │   │   └── validation.js
│   │   ├── services/
│   │   │   ├── paymentService.js
│   │   │   ├── emailService.js
│   │   │   └── inventoryService.js
│   │   └── utils/
│   │       └── helpers.js
│   ├── uploads/
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── common/
│   │   │   ├── Product/
│   │   │   ├── Cart/
│   │   │   ├── Checkout/
│   │   │   └── Admin/
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   ├── ProductList.jsx
│   │   │   ├── ProductDetail.jsx
│   │   │   ├── Cart.jsx
│   │   │   ├── Checkout.jsx
│   │   │   ├── Orders.jsx
│   │   │   └── Admin/
│   │   ├── context/
│   │   │   ├── AuthContext.jsx
│   │   │   └── CartContext.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   └── App.jsx
│   └── package.json
└── README.md
```

### Step 2: Design Database Schema

**Product Model:**

```javascript
// models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 200
  },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  description: {
    type: String,
    required: true,
    maxlength: 5000
  },
  price: {
    type: Number,
    required: true,
    min: 0
  },
  compareAtPrice: {
    type: Number,
    min: 0
  },
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category',
    required: true
  },
  images: [{
    url: String,
    publicId: String,
    altText: String
  }],
  stock: {
    type: Number,
    required: true,
    min: 0,
    default: 0
  },
  sku: {
    type: String,
    unique: true,
    sparse: true
  },
  featured: {
    type: Boolean,
    default: false
  },
  averageRating: {
    type: Number,
    default: 0,
    min: 0,
    max: 5
  },
  numReviews: {
    type: Number,
    default: 0
  },
  isActive: {
    type: Boolean,
    default: true
  },
  specifications: {
    type: Map,
    of: String
  },
  tags: [String],
  soldCount: {
    type: Number,
    default: 0
  }
}, {
  timestamps: true
});

// Index for search and filtering
productSchema.index({ name: 'text', description: 'text', tags: 'text' });
productSchema.index({ category: 1, price: 1 });
productSchema.index({ featured: 1, soldCount: -1 });

module.exports = mongoose.model('Product', productSchema);
```

**Order Model:**

```javascript
// models/Order.js
const mongoose = require('mongoose');

const orderItemSchema = new mongoose.Schema({
  product: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Product',
    required: true
  },
  name: String,
  image: String,
  price: Number,
  quantity: {
    type: Number,
    required: true,
    min: 1
  }
});

const orderSchema = new mongoose.Schema({
  orderNumber: {
    type: String,
    required: true,
    unique: true
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  items: [orderItemSchema],
  shippingAddress: {
    fullName: String,
    address: String,
    city: String,
    postalCode: String,
    country: String,
    phone: String
  },
  paymentMethod: {
    type: String,
    required: true,
    enum: ['card', 'paypal']
  },
  paymentResult: {
    id: String,
    status: String,
    email: String
  },
  itemsPrice: {
    type: Number,
    required: true,
    default: 0
  },
  taxPrice: {
    type: Number,
    required: true,
    default: 0
  },
  shippingPrice: {
    type: Number,
    required: true,
    default: 0
  },
  totalPrice: {
    type: Number,
    required: true,
    default: 0
  },
  isPaid: {
    type: Boolean,
    default: false
  },
  paidAt: Date,
  isDelivered: {
    type: Boolean,
    default: false
  },
  deliveredAt: Date,
  status: {
    type: String,
    enum: ['pending', 'processing', 'shipped', 'delivered', 'cancelled'],
    default: 'pending'
  },
  trackingNumber: String
}, {
  timestamps: true
});

// Generate order number before saving
orderSchema.pre('save', async function(next) {
  if (!this.orderNumber) {
    const count = await mongoose.model('Order').countDocuments();
    this.orderNumber = `ORD-${Date.now()}-${count + 1}`;
  }
  next();
});

module.exports = mongoose.model('Order', orderSchema);
```

**Cart Model:**

```javascript
// models/Cart.js
const mongoose = require('mongoose');

const cartItemSchema = new mongoose.Schema({
  product: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Product',
    required: true
  },
  quantity: {
    type: Number,
    required: true,
    min: 1
  },
  price: Number
});

const cartSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    unique: true
  },
  items: [cartItemSchema],
  totalPrice: {
    type: Number,
    default: 0
  }
}, {
  timestamps: true
});

// Calculate total price before saving
cartSchema.pre('save', function(next) {
  this.totalPrice = this.items.reduce((total, item) => {
    return total + (item.price * item.quantity);
  }, 0);
  next();
});

module.exports = mongoose.model('Cart', cartSchema);
```

### Step 3: Implement Shopping Cart

```javascript
// controllers/cartController.js
const Cart = require('../models/Cart');
const Product = require('../models/Product');

// Get cart
exports.getCart = async (req, res) => {
  try {
    let cart = await Cart.findOne({ user: req.userId })
      .populate('items.product', 'name price images stock');
    
    if (!cart) {
      cart = new Cart({ user: req.userId, items: [] });
      await cart.save();
    }
    
    res.json({ cart });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Add item to cart
exports.addToCart = async (req, res) => {
  try {
    const { productId, quantity = 1 } = req.body;
    
    // Check product exists and has stock
    const product = await Product.findById(productId);
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }
    
    if (product.stock < quantity) {
      return res.status(400).json({ error: 'Insufficient stock' });
    }
    
    // Get or create cart
    let cart = await Cart.findOne({ user: req.userId });
    if (!cart) {
      cart = new Cart({ user: req.userId, items: [] });
    }
    
    // Check if product already in cart
    const existingItemIndex = cart.items.findIndex(
      item => item.product.toString() === productId
    );
    
    if (existingItemIndex > -1) {
      // Update quantity
      const newQuantity = cart.items[existingItemIndex].quantity + quantity;
      if (newQuantity > product.stock) {
        return res.status(400).json({ error: 'Insufficient stock' });
      }
      cart.items[existingItemIndex].quantity = newQuantity;
    } else {
      // Add new item
      cart.items.push({
        product: productId,
        quantity,
        price: product.price
      });
    }
    
    await cart.save();
    await cart.populate('items.product', 'name price images stock');
    
    res.json({
      message: 'Item added to cart',
      cart
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Update cart item quantity
exports.updateCartItem = async (req, res) => {
  try {
    const { productId, quantity } = req.body;
    
    const cart = await Cart.findOne({ user: req.userId });
    if (!cart) {
      return res.status(404).json({ error: 'Cart not found' });
    }
    
    const itemIndex = cart.items.findIndex(
      item => item.product.toString() === productId
    );
    
    if (itemIndex === -1) {
      return res.status(404).json({ error: 'Item not in cart' });
    }
    
    // Check stock
    const product = await Product.findById(productId);
    if (quantity > product.stock) {
      return res.status(400).json({ error: 'Insufficient stock' });
    }
    
    if (quantity <= 0) {
      // Remove item
      cart.items.splice(itemIndex, 1);
    } else {
      // Update quantity
      cart.items[itemIndex].quantity = quantity;
    }
    
    await cart.save();
    await cart.populate('items.product', 'name price images stock');
    
    res.json({
      message: 'Cart updated',
      cart
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Remove item from cart
exports.removeFromCart = async (req, res) => {
  try {
    const { productId } = req.params;
    
    const cart = await Cart.findOne({ user: req.userId });
    if (!cart) {
      return res.status(404).json({ error: 'Cart not found' });
    }
    
    cart.items = cart.items.filter(
      item => item.product.toString() !== productId
    );
    
    await cart.save();
    await cart.populate('items.product', 'name price images stock');
    
    res.json({
      message: 'Item removed from cart',
      cart
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Clear cart
exports.clearCart = async (req, res) => {
  try {
    const cart = await Cart.findOne({ user: req.userId });
    if (cart) {
      cart.items = [];
      await cart.save();
    }
    
    res.json({ message: 'Cart cleared' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Step 4: Integrate Stripe Payment

```javascript
// services/paymentService.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

class PaymentService {
  // Create payment intent
  async createPaymentIntent(amount, currency = 'usd', metadata = {}) {
    try {
      const paymentIntent = await stripe.paymentIntents.create({
        amount: Math.round(amount * 100), // Convert to cents
        currency,
        metadata,
        automatic_payment_methods: {
          enabled: true
        }
      });
      
      return paymentIntent;
    } catch (error) {
      throw new Error(`Payment intent creation failed: ${error.message}`);
    }
  }
  
  // Verify payment
  async verifyPayment(paymentIntentId) {
    try {
      const paymentIntent = await stripe.paymentIntents.retrieve(paymentIntentId);
      return paymentIntent.status === 'succeeded';
    } catch (error) {
      throw new Error(`Payment verification failed: ${error.message}`);
    }
  }
  
  // Create refund
  async createRefund(paymentIntentId, amount) {
    try {
      const refund = await stripe.refunds.create({
        payment_intent: paymentIntentId,
        amount: amount ? Math.round(amount * 100) : undefined
      });
      
      return refund;
    } catch (error) {
      throw new Error(`Refund failed: ${error.message}`);
    }
  }
  
  // Handle webhook
  async handleWebhook(payload, signature) {
    try {
      const event = stripe.webhooks.constructEvent(
        payload,
        signature,
        process.env.STRIPE_WEBHOOK_SECRET
      );
      
      return event;
    } catch (error) {
      throw new Error(`Webhook error: ${error.message}`);
    }
  }
}

module.exports = new PaymentService();
```

```javascript
// controllers/orderController.js
const Order = require('../models/Order');
const Cart = require('../models/Cart');
const Product = require('../models/Product');
const paymentService = require('../services/paymentService');
const emailService = require('../services/emailService');

// Create order
exports.createOrder = async (req, res) => {
  try {
    const { shippingAddress, paymentMethod } = req.body;
    
    // Get user's cart
    const cart = await Cart.findOne({ user: req.userId })
      .populate('items.product');
    
    if (!cart || cart.items.length === 0) {
      return res.status(400).json({ error: 'Cart is empty' });
    }
    
    // Verify stock availability
    for (const item of cart.items) {
      if (item.product.stock < item.quantity) {
        return res.status(400).json({
          error: `Insufficient stock for ${item.product.name}`
        });
      }
    }
    
    // Calculate prices
    const itemsPrice = cart.totalPrice;
    const taxPrice = itemsPrice * 0.1; // 10% tax
    const shippingPrice = itemsPrice > 100 ? 0 : 10; // Free shipping over $100
    const totalPrice = itemsPrice + taxPrice + shippingPrice;
    
    // Create payment intent
    const paymentIntent = await paymentService.createPaymentIntent(
      totalPrice,
      'usd',
      { userId: req.userId.toString() }
    );
    
    // Create order
    const order = new Order({
      user: req.userId,
      items: cart.items.map(item => ({
        product: item.product._id,
        name: item.product.name,
        image: item.product.images[0]?.url,
        price: item.product.price,
        quantity: item.quantity
      })),
      shippingAddress,
      paymentMethod,
      itemsPrice,
      taxPrice,
      shippingPrice,
      totalPrice,
      paymentResult: {
        id: paymentIntent.id,
        status: paymentIntent.status
      }
    });
    
    await order.save();
    
    res.status(201).json({
      message: 'Order created successfully',
      order,
      clientSecret: paymentIntent.client_secret
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Confirm payment and update order
exports.confirmPayment = async (req, res) => {
  try {
    const { orderId, paymentIntentId } = req.body;
    
    // Verify payment
    const isPaymentSuccessful = await paymentService.verifyPayment(paymentIntentId);
    
    if (!isPaymentSuccessful) {
      return res.status(400).json({ error: 'Payment not successful' });
    }
    
    // Update order
    const order = await Order.findById(orderId);
    if (!order) {
      return res.status(404).json({ error: 'Order not found' });
    }
    
    order.isPaid = true;
    order.paidAt = Date.now();
    order.status = 'processing';
    await order.save();
    
    // Update product stock and sold count
    for (const item of order.items) {
      await Product.findByIdAndUpdate(item.product, {
        $inc: {
          stock: -item.quantity,
          soldCount: item.quantity
        }
      });
    }
    
    // Clear cart
    await Cart.findOneAndUpdate(
      { user: req.userId },
      { $set: { items: [] } }
    );
    
    // Send confirmation email
    await emailService.sendOrderConfirmation(order);
    
    res.json({
      message: 'Payment confirmed successfully',
      order
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Get user orders
exports.getUserOrders = async (req, res) => {
  try {
    const orders = await Order.find({ user: req.userId })
      .sort({ createdAt: -1 })
      .populate('items.product', 'name images');
    
    res.json({ orders });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Step 5: Create Frontend Cart Component

```jsx
// components/Cart/Cart.jsx
import React, { useContext, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { CartContext } from '../../context/CartContext';
import { Trash2, Plus, Minus } from 'lucide-react';

const Cart = () => {
  const { cart, updateQuantity, removeItem, loading } = useContext(CartContext);
  const navigate = useNavigate();
  
  const handleQuantityChange = async (productId, newQuantity) => {
    if (newQuantity < 1) return;
    await updateQuantity(productId, newQuantity);
  };
  
  const handleCheckout = () => {
    if (cart?.items.length > 0) {
      navigate('/checkout');
    }
  };
  
  if (loading) {
    return <div className="text-center py-8">Loading...</div>;
  }
  
  if (!cart || cart.items.length === 0) {
    return (
      <div className="container mx-auto px-4 py-8">
        <div className="text-center">
          <h2 className="text-2xl font-bold mb-4">Your cart is empty</h2>
          <button
            onClick={() => navigate('/products')}
            className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
          >
            Continue Shopping
          </button>
        </div>
      </div>
    );
  }
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">Shopping Cart</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        {/* Cart Items */}
        <div className="lg:col-span-2">
          {cart.items.map((item) => (
            <div
              key={item.product._id}
              className="flex items-center gap-4 bg-white p-4 rounded-lg shadow-md mb-4"
            >
              <img
                src={item.product.images[0]?.url}
                alt={item.product.name}
                className="w-24 h-24 object-cover rounded"
              />
              
              <div className="flex-1">
                <h3 className="font-semibold text-lg">{item.product.name}</h3>
                <p className="text-gray-600">${item.price.toFixed(2)}</p>
                
                <div className="flex items-center gap-2 mt-2">
                  <button
                    onClick={() => handleQuantityChange(item.product._id, item.quantity - 1)}
                    className="p-1 rounded hover:bg-gray-100"
                  >
                    <Minus size={16} />
                  </button>
                  
                  <span className="px-3">{item.quantity}</span>
                  
                  <button
                    onClick={() => handleQuantityChange(item.product._id, item.quantity + 1)}
                    className="p-1 rounded hover:bg-gray-100"
                    disabled={item.quantity >= item.product.stock}
                  >
                    <Plus size={16} />
                  </button>
                </div>
              </div>
              
              <div className="text-right">
                <p className="font-bold text-lg">
                  ${(item.price * item.quantity).toFixed(2)}
                </p>
                
                <button
                  onClick={() => removeItem(item.product._id)}
                  className="text-red-500 hover:text-red-700 mt-2"
                >
                  <Trash2 size={20} />
                </button>
              </div>
            </div>
          ))}
        </div>
        
        {/* Order Summary */}
        <div className="lg:col-span-1">
          <div className="bg-white p-6 rounded-lg shadow-md sticky top-4">
            <h2 className="text-xl font-bold mb-4">Order Summary</h2>
            
            <div className="space-y-2 mb-4">
              <div className="flex justify-between">
                <span>Subtotal:</span>
                <span>${cart.totalPrice.toFixed(2)}</span>
              </div>
              
              <div className="flex justify-between">
                <span>Shipping:</span>
                <span>{cart.totalPrice > 100 ? 'FREE' : '$10.00'}</span>
              </div>
              
              <div className="flex justify-between">
                <span>Tax (10%):</span>
                <span>${(cart.totalPrice * 0.1).toFixed(2)}</span>
              </div>
              
              <div className="border-t pt-2 mt-2">
                <div className="flex justify-between font-bold text-lg">
                  <span>Total:</span>
                  <span>
                    ${(
                      cart.totalPrice +
                      (cart.totalPrice > 100 ? 0 : 10) +
                      cart.totalPrice * 0.1
                    ).toFixed(2)}
                  </span>
                </div>
              </div>
            </div>
            
            <button
              onClick={handleCheckout}
              className="w-full bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 font-semibold"
            >
              Proceed to Checkout
            </button>
            
            <button
              onClick={() => navigate('/products')}
              className="w-full mt-2 border border-gray-300 py-3 rounded-lg hover:bg-gray-50"
            >
              Continue Shopping
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Cart;
```

---

## Technology Stack

### Backend
- **Node.js + Express**: Server framework
- **MongoDB + Mongoose**: Database
- **Stripe API**: Payment processing
- **JWT**: Authentication
- **Cloudinary**: Image storage
- **Nodemailer**: Email service
- **Express-validator**: Input validation

### Frontend
- **React**: UI library
- **React Router**: Navigation
- **Context API/Redux**: State management
- **Stripe.js**: Payment UI
- **Axios**: HTTP client
- **Tailwind CSS**: Styling
- **React Query**: Data fetching

### DevOps
- **Docker**: Containerization
- **PM2**: Process manager
- **Nginx**: Web server

---

## Testing Checklist

- [ ] User authentication working
- [ ] Product catalog displays correctly
- [ ] Search and filters functional
- [ ] Add to cart working
- [ ] Cart updates in real-time
- [ ] Stock validation working
- [ ] Checkout flow complete
- [ ] Payment processing secure
- [ ] Order confirmation sent
- [ ] Admin dashboard functional
- [ ] Inventory updates correctly
- [ ] Email notifications working
- [ ] Mobile responsive
- [ ] Performance optimized

---

## Additional Challenges

### Easy
- Add product wishlist
- Implement product quick view
- Add related products section
- Create size/color variants

### Medium
- Build coupon discount system
- Add product reviews and ratings
- Implement advanced search with Algolia
- Create abandoned cart recovery
- Add multi-currency support
- Build loyalty points system

### Hard
- Implement recommendation engine
- Add live chat support
- Create subscription products
- Build multi-vendor marketplace
- Implement advanced analytics
- Add AI-powered product search
- Create mobile app with React Native
- Implement internationalization (i18n)

---

## Common Pitfalls

**Insecure payment handling**: Never store card details, use Stripe/PayPal  
**No inventory management**: Implement real-time stock tracking  
**Poor checkout UX**: Minimize steps and provide guest checkout  
**Missing order notifications**: Send confirmation emails  
**No error handling**: Gracefully handle payment failures  
**Ignoring SEO**: Optimize product pages for search engines  

---

## Resources

### Documentation
- [Stripe Documentation](https://stripe.com/docs)
- [PayPal Developer](https://developer.paypal.com/)
- [PCI Compliance](https://www.pcisecuritystandards.org/)

### Tutorials
- [Building E-commerce with Stripe](https://stripe.com/docs/payments/accept-a-payment)
- [E-commerce Best Practices](https://www.shopify.com/blog/ecommerce-website-design)

---

## License

This project idea is part of the [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library) and is available under the MIT License.
