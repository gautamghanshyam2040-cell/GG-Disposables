<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>GG Disposable Products - Firebase Enabled</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
body{margin:0;font-family:Arial,sans-serif;background:#f4f6f9;}
nav{background:#0f172a;color:white;display:flex;justify-content:space-between;align-items:center;padding:15px 30px;position:sticky;top:0;z-index:1000;}
nav a{color:white;text-decoration:none;margin-left:20px;font-weight:500;}
nav a:hover{color:#38bdf8;}
header{background:linear-gradient(rgba(0,0,0,0.6),rgba(0,0,0,0.6)),url('https://www.nivabupa.com/uploads/the_glut_of_paper_products_a0c98d91d2.jpg');background-size:cover;color:white;text-align:center;padding:120px 20px;}
header h1{font-size:48px;margin-bottom:10px;}
.btn{background:#0ea5e9;color:white;padding:12px 25px;border-radius:30px;text-decoration:none;display:inline-block;margin-top:20px;transition:0.3s;}
.btn:hover{background:#0284c7;}
.search-box{text-align:center;margin:20px;}
.search-box input{width:300px;max-width:90%;padding:10px;border-radius:6px;border:1px solid #ccc;}
.category-filter{text-align:center;margin:20px;}
.category-filter button{margin:5px;padding:7px 15px;border:none;background:#0ea5e9;color:white;border-radius:6px;cursor:pointer;transition:0.3s;}
.category-filter button:hover{background:#0284c7;}
.products-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:20px;padding:30px;}
.product-card{background:white;border-radius:12px;padding:15px;text-align:center;box-shadow:0 5px 15px rgba(0,0,0,0.1);transition:0.3s;}
.product-card:hover{transform:translateY(-8px);box-shadow:0 15px 30px rgba(0,0,0,0.2);}
.product-card img{width:100%;height:170px;object-fit:contain;cursor:pointer;}
.product-card input{width:60px;padding:5px;margin-top:5px;}
.price{color:#22c55e;font-weight:bold;margin:8px 0;}
#cart{padding:20px;background:white;max-width:600px;margin:30px auto;border-radius:12px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
#cartItems div{display:flex;justify-content:space-between;margin:5px 0;}
#cart h2{text-align:center;}
#cart button.checkout-btn{background:#22c55e;color:white;padding:10px 20px;border:none;border-radius:6px;margin-top:10px;cursor:pointer;transition:0.3s;}
#cart button.checkout-btn:hover{background:#16a34a;}
#downloadInvoiceBtn{background:#0ea5e9;color:white;padding:10px 20px;border:none;border-radius:6px;margin-top:10px;cursor:pointer;transition:0.3s;}
#downloadInvoiceBtn:hover{background:#0284c7;}
#lightbox{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.8);display:none;justify-content:center;align-items:center;z-index:999;}
#lightbox img{max-width:90%;max-height:90%;}
footer{background:#0f172a;color:white;text-align:center;padding:20px;margin-top:40px;}
</style>
</head>
<body>

<nav>
<div style="display:flex;align-items:center;gap:10px">
<img src="https://cdn-icons-png.flaticon.com/512/3075/3075977.png" width="35">
<h3>GG Disposable</h3>
</div>
<div>
<a href="#">Home</a>
<a href="#products">Products</a>
<a href="#cart">Cart (<span id="cartCount">0</span>)</a>
<a href="#contact">Contact</a>
<a href="admin.html">Admin Dashboard</a>
</div>
</nav>

<header>
<h1>GG Disposable Products</h1>
<p>Disposable Plates · Bowls · Spoons · Glasses</p>
<a class="btn" href="#products">Shop Now</a>
</header>

<div class="search-box">
<input type="text" id="search" placeholder="Search product...">
</div>

<div class="category-filter">
<button onclick="filterProducts('all')">All</button>
<button onclick="filterProducts('plates')">Plates</button>
<button onclick="filterProducts('spoons')">Spoons</button>
<button onclick="filterProducts('bowls')">Bowls</button>
<button onclick="filterProducts('glasses')">Glasses</button>
</div>

<section id="products">
<div class="products-grid">
<!-- Product cards same as before -->
</div>
</section>

<section id="cart">
<h2>Your Cart</h2>
<div id="cartItems"></div>
<h3>Total ₹<span id="cartTotal">0</span></h3>
<button class="checkout-btn" onclick="checkout()">Checkout</button>
<button class="checkout-btn" onclick="payNow()">Pay Online</button>
<button id="downloadInvoiceBtn" onclick="downloadInvoice()" style="display:none;">Download Invoice</button>
</section>

<div id="lightbox"><img src=""></div>

<footer>
<p>© 2026 GG Disposable Products</p>
<p>Owner: Ghanshyam Gautam</p>
<p>Email: gautamghanshyam2040@gmail.com</p>
<p>📞 7291074410</p>
</footer>

<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
<script>
// ===== FIREBASE CONFIG =====
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  databaseURL: "YOUR_DB_URL",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MSG_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ===== CART SYSTEM =====
let cart = [];
const downloadInvoiceBtn = document.getElementById("downloadInvoiceBtn");

function addToCart(name, price, btn){
    let qty = parseInt(btn.parentElement.querySelector("input").value);
    if(qty<=0){ alert("Select quantity first!"); return; }
    cart.push({name, price, qty});
    updateCart();
}

function updateCart(){
    let html=""; let total=0; let count=0;
    cart.forEach((item,index)=>{
        total += item.price*item.qty;
        count += item.qty;
        html += `<div>${item.name} x ${item.qty} - ₹${item.price*item.qty} <button onclick=\"removeItem(${index})\">Remove</button></div>`;
    });
    document.getElementById("cartItems").innerHTML = html;
    document.getElementById("cartTotal").innerText = total;
    document.getElementById("cartCount").innerText = count;
}

function removeItem(index){ cart.splice(index,1); updateCart(); }
function filterProducts(category){ document.querySelectorAll('.product-card').forEach(card=>{ card.style.display = (category=='all'||card.dataset.category==category)?'block':'none'; });}

// ===== LIGHTBOX =====
document.querySelectorAll(".product-card img").forEach(img=>{ img.onclick=function(){ document.getElementById("lightbox").style.display="flex"; document.querySelector("#lightbox img").src=this.src; } });
document.getElementById("lightbox").onclick=function(){ this.style.display="none"; }

// ===== CHECKOUT & SAVE TO FIREBASE =====
function checkout(){
    if(cart.length==0){ alert("Cart empty"); return; }
    let name=prompt("Enter your name"); if(!name){ alert("Name is required"); return; }
    let mobile=prompt("Enter mobile number"); if(!mobile){ alert("Mobile number is required"); return; }
    let address=prompt("Enter delivery address"); if(!address){ alert("Address is required"); return; }
    let total=0; cart.forEach(item=> total+=item.price*item.qty);

    let orderData={name,mobile,address,products:[...cart],total,date:new Date().toLocaleString()};
    db.ref('orders').push(orderData);

    window.lastInvoiceData = orderData;
    generateInvoice(name,mobile,address,cart,total);
    downloadInvoiceBtn.style.display = "inline-block";

    cart=[]; updateCart();
    document.querySelectorAll('.product-card input').forEach(i=>i.value=0);
    alert("Order placed successfully!");
}

function generateInvoice(name,mobile,address,products,total){
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    let y=20; doc.setFontSize(18); doc.text("GG Disposable Products",105,y,null,null,"center"); y+=10;
    doc.setFontSize(12); doc.text("Customer: "+name,14,y); y+=7;
    doc.text("Mobile: "+mobile,14,y); y+=7;
    doc.text("Address: "+address,14,y); y+=10;
    doc.text("Item",14,y); doc.text("Qty",100,y); doc.text("Price",130,y); doc.text("Amount",160,y); y+=6;
    products.forEach(p=>{ doc.text(p.name,14,y); doc.text(String(p.qty),100,y); doc.text(String(p.price),130,y); doc.text(String(p.qty*p.price),160,y); y+=6; });
    y+=5; doc.text("Total ₹"+total,150,y); doc.save("GG_Disposable_Invoice.pdf");
}

function downloadInvoice(){ if(!window.lastInvoiceData){ alert("No invoice available!"); return; } generateInvoice(window.lastInvoiceData.name,window.lastInvoiceData.mobile,window.lastInvoiceData.address,window.lastInvoiceData.products,window.lastInvoiceData.total); }

function payNow(){ let total=document.getElementById("cartTotal").innerText*100; if(total==0){ alert("Cart empty"); return; } var options={ key:"rzp_test_xxxxxxxxxxxxx", amount:total, currency:"INR", name:"GG Disposable", description:"Order Payment", handler:function(res){ alert("Payment Successful!"); } }; var rzp=new Razorpay(options); rzp.open(); }
</script>
</body>
</html>
