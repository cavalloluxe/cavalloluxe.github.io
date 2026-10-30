CAVALLOLUXE — FIREBASE MIGRATION SETUP

WHAT THESE FILES DO
- admin.html and index.html use Firebase Realtime Database and Firebase Storage.
- Admin writes shared catalogue, orders, coupons, stock, and CMS data under /store.
- Storefront reads only public products, coupons, and CMS nodes, and creates orders under /store/orders.
- Gemini AI runs in the clAi Firebase Cloud Function. No Gemini key belongs in either HTML file.

IMPORTANT SECURITY
1. Revoke the Gemini API key that was pasted into chat. Create a fresh key in Google AI Studio.
2. Never put the Gemini key in admin.html, index.html, or any public repository.
3. Do not publish service-account.json. It is private server credentials.

FIREBASE CONSOLE SETUP
1. Open project cavalloluxe-acbdf in Firebase Console.
2. Authentication > Sign-in method: enable Email/Password.
3. Authentication > Users: create the admin account you will use to sign into admin.html.
4. Realtime Database: publish the rules from database.rules.json.
5. Storage: enable Firebase Storage and publish storage.rules.
6. In Authentication > Settings > Authorized domains, add your website's GitHub Pages/custom domain.
7. Copy the admin user's UID from Authentication > Users.

ADMIN PRIVILEGES / STORAGE CLAIM
The admin panel checks /admins/{uid} in Realtime Database. Storage uploads additionally require the admin custom claim.
- From the Firebase project folder run: cd functions && npm install
- Download a service account JSON from Firebase Project Settings > Service accounts. Save it as functions/service-account.json. Keep it private and never upload it to GitHub.
- Edit functions/set-admin-claim.js and replace PASTE_YOUR_ADMIN_USER_UID_HERE with the admin UID.
- From the functions folder run: node set-admin-claim.js
- Sign out and sign back into admin.html so Firebase refreshes the ID token.

GEMINI CLOUD FUNCTION
1. Install Firebase CLI: npm install -g firebase-tools
2. Sign in: firebase login
3. From the project folder, select the existing project: firebase use cavalloluxe-acbdf
4. Put the functions folder from this package in your Firebase project root.
5. Run: cd functions && npm install && cd ..
6. Store the NEW Gemini key as a server secret: firebase functions:secrets:set GEMINI_API_KEY
   Paste the new key only into the CLI prompt.
7. Deploy: firebase deploy --only functions
8. The AI endpoint is configured for us-central1 and is named clAi.

FIRST DATA LOAD
- On first successful admin login, the panel seeds Firebase with the built-in product catalogue if /store/products is empty.
- New edits made in admin.html then sync to index.html through Firebase.
- Existing records that currently exist only in your old Supabase project are NOT automatically copied by this HTML migration. Export them from Supabase before shutting it down, then import/convert them into the Firebase /store structure. Do not delete your old Supabase project until you verify products, orders, images, and settings in Firebase.

EXPECTED FIREBASE DATA SHAPE
/store/products/{productId}: {id,name,cat,price,old,img,imgs,cols,desc,stock,on,flags,offer,rating,reviewText,sort}
/store/orders/{orderId}: order objects created by the storefront or admin panel
/store/coupons/{couponCode}: {code,type,value,min,on,used}
/store/cms: {music:[],banners:[],categories:[],reviews:[],branding:{...}}
/store/stockMoves: array of stock movement records
/store/seq: next admin order number
/admins/{uid}: true

TEST CHECKLIST
- Sign into admin.html using the Firebase admin account.
- Add a test product with a distinctive name and image; save it.
- Open index.html on the published domain and verify the product appears.
- Edit its price/visibility in admin and verify storefront updates.
- Upload a product photo and a CMS banner to confirm Storage access.
- Place a test COD order and verify it appears in Admin > Orders and stock decreases.
- Ask CL AI a question and verify the secure function replies. CL AI is read-only in this migration: it analyses store data and explains actions, but it does not directly edit records.

NOTE
Firebase web config values in the HTML identify the project; database and storage rules plus admin authentication are what protect the data. Do not use public read/write rules for the whole database.
