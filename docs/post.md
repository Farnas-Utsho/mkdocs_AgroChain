## Imporatnce of Product Listing Management
The "Product Listing Management" feature is integral to the AgroChain app, offering a platform where farmers, wholesalers, and retailers can showcase their agricultural products. It serves as a digital marketplace, providing users with visibility and access to a wide range of agricultural commodities. Users can create detailed listings, including product descriptions, images, pricing, and quantities, enhancing transparency and trust in the marketplace. This feature enables efficient inventory management, allowing wholesalers and retailers to track their available stock and update listings accordingly. Farmers can leverage the platform to reach a broader audience and increase their sales opportunities, thereby boosting their income and livelihoods. Buyers benefit from a diverse selection of products, real-time updates on availability, and the ability to make informed purchasing decisions. The feedback loop within the feature enables users to receive reviews and ratings, fostering continuous improvement and customer satisfaction. Aggregated data from product listings offers valuable insights into market trends, demand patterns, and pricing dynamics, empowering users with actionable information for strategic decision-making. Ultimately, the "Product Listing Management" feature contributes to the growth, efficiency, and sustainability of the agricultural ecosystem within the AgroChain app.

## Here's a breakdown of the key activities happening in the `CreatePostActivity` class:

1. **UI Initialization**: In the `onCreate` method, the UI elements such as spinners, text views, edit texts, image view, and button are initialized. Data loading methods (`loadSeasons()` and `loadProducts()`) are called to populate the spinners with data.
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_create_post);

    storageReferance = FirebaseStorage.getInstance().getReference();

    spinnerSeason = findViewById(R.id.spinnerSeason);
    spinnerProduct = findViewById(R.id.spinnerProduct);
    textViewPrice = findViewById(R.id.textViewPrice);

    editTextQuantity = findViewById(R.id.editTextQuantity);
    editTextReleaseDate = findViewById(R.id.editTextReleaseDate);
    editTextNote = findViewById(R.id.editTextNote);
    imageViewPostImage = findViewById(R.id.imageViewPostImage);
    buttonSubmitPost = findViewById(R.id.buttonSubmitPost);

    loadSeasons();

    editTextReleaseDate.setOnClickListener(v -> showDatePickerDialog());
    imageViewPostImage.setOnClickListener(view -> {
        Intent intent = new Intent(Intent.ACTION_PICK);
        intent.setType("image/*");
        activityResultLauncher.launch(intent);
    });

    buttonSubmitPost.setOnClickListener(view -> {
        if (image != null) {
            UploadImage(image);
        } else {
            Toast.makeText(CreatePostActivity.this, "Select an image", Toast.LENGTH_SHORT).show();
        }
    });

    String userID = getUserID();
    if (userID != null) {
        Toast.makeText(this, "User ID: " + userID, Toast.LENGTH_SHORT).show();
    } else {
        Toast.makeText(this, "User ID is missing", Toast.LENGTH_SHORT).show();
    }
}

```

2. **Image Selection**: When the user clicks on the `imageViewPostImage`, an intent is created to pick an image from the device gallery. The selected image is then displayed in the `imageViewPostImage`.

```
imageViewPostImage.setOnClickListener(view -> {
    Intent intent = new Intent(Intent.ACTION_PICK);
    intent.setType("image/*");
    activityResultLauncher.launch(intent);
});



```

3. **Date Selection**: When the user clicks on the `editTextReleaseDate`, a date picker dialog is displayed, allowing the user to select a release date for the post.
```
editTextReleaseDate.setOnClickListener(v -> showDatePickerDialog());

```
4. **Firebase Data Loading**: The `loadSeasons()` and `loadProducts()` methods fetch data from Firestore to populate the spinners with seasons and products dynamically.

```
private void loadSeasons() {
    FirebaseFirestore db = FirebaseFirestore.getInstance();
    db.collection("GOV").document("GOVA-CyUsLz").collection("PRODUCT")
            .get().addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    List<String> seasons = new ArrayList<>();
                    for (QueryDocumentSnapshot document : task.getResult()) {
                        String category = document.getString("category");
                        if (!seasons.contains(category)) {
                            seasons.add(category);
                        }
                    }
                    ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, seasons);
                    adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
                    spinnerSeason.setAdapter(adapter);

                    spinnerSeason.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
                        @Override
                        public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                            String selectedSeason = adapter.getItem(position);
                            if (selectedSeason != null) {
                                loadProducts(selectedSeason);
                            }
                        }

                        @Override
                        public void onNothingSelected(AdapterView<?> parent) {
                        }
                    });
                } else {
                    Log.w(TAG, "Error getting documents.", task.getException());
                }
            });
}

private void loadProducts(String season) {
    FirebaseFirestore db = FirebaseFirestore.getInstance();
    db.collection("GOV").document("GOVA-CyUsLz").collection("PRODUCT")
            .whereEqualTo("category", season)
            .get()
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    List<String> products = new ArrayList<>();
                    Map<String, String> productPriceMap = new HashMap<>();
                    for (QueryDocumentSnapshot document : task.getResult()) {
                        String productName = document.getString("name");
                        String price = document.getString("price");
                        products.add(productName);
                        productPriceMap.put(productName, price);
                    }

                    if (!products.isEmpty()) {
                        ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, products);
                        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
                        spinnerProduct.setAdapter(adapter);

                        spinnerProduct.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
                            @Override
                            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                                String selectedProduct = adapter.getItem(position);
                                if (selectedProduct != null && productPriceMap.containsKey(selectedProduct)) {
                                    textViewPrice.setText(productPriceMap.get(selectedProduct));
                                }
                            }

                            @Override
                            public void onNothingSelected(AdapterView<?> parent) {
                                textViewPrice.setText("");
                            }
                        });
                    } else {
                        Toast.makeText(CreatePostActivity.this, "No products found for the selected season.", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Log.w(TAG, "Error getting documents: ", task.getException());
                }
            });
}

```


5. **Image Upload**: When the user selects an image, it is uploaded to Firebase Storage using the `UploadImage()` method. A progress dialog is displayed during the upload process.

6. **Post Submission**: When the user clicks on the `buttonSubmitPost`, the `submitPost()` method is called to gather post data (category, product, quantity, release date, note, and image URL) and store it in Firestore under the appropriate collection based on the user's ID.

```
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_create_post);

        storageReferance=FirebaseStorage.getInstance().getReference();

        spinnerSeason = findViewById(R.id.spinnerSeason);
        spinnerProduct = findViewById(R.id.spinnerProduct);
        textViewPrice = findViewById(R.id.textViewPrice);

        editTextQuantity = findViewById(R.id.editTextQuantity);
        editTextReleaseDate = findViewById(R.id.editTextReleaseDate);
        editTextNote = findViewById(R.id.editTextNote);
        imageViewPostImage = findViewById(R.id.imageViewPostImage);
        buttonSubmitPost = findViewById(R.id.buttonSubmitPost);

        loadSeasons();

        editTextReleaseDate.setOnClickListener(v -> showDatePickerDialog());
        imageViewPostImage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(Intent.ACTION_PICK);
                intent.setType("image/*");
                activityResultLauncher.launch(intent);
            }
        });
        buttonSubmitPost.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if(image!=null){
                    UploadImage(image);
                }else{
                    Toast.makeText(CreatePostActivity.this, "Select an image", Toast.LENGTH_SHORT).show();
                }
            }
        });


        String userID = getUserID();
        if (userID != null) {
            Toast.makeText(this, "User ID: " + userID, Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "User ID is missing", Toast.LENGTH_SHORT).show();
        }
    }

```

7. **User ID Retrieval**: The `getUserID()` method retrieves the user's ID from SharedPreferences.

8. **Collection Name Determination**: The `determineCollectionName()` method determines the Firestore collection name based on the user's ID prefix (e.g., FAR, GOV, WHO, RET, CUS).

Overall, this activity allows users to create and submit posts with images, which are stored in Firestore and Firebase Storage. It interacts with Firebase for data storage and retrieval. Additionally, it handles user interactions for selecting images and dates.


#

### SendWarning() method sends a warning notification to the farmer, which is essential for alerting them about potential issues or concerns raised by other users. It creates a notification document in the farmer's collection in Firestore, containing the message, timestamp, and read status. Firestore operations are asynchronous and handle success and failure cases, ensuring reliable notification delivery.

```
private void sendWarning(String farmerId, String comment) {
    Map<String, Object> notificationData = new HashMap<>();
    notificationData.put("message", comment);
    notificationData.put("timestamp", new Date());
    notificationData.put("read", false);

    FirebaseFirestore db = FirebaseFirestore.getInstance();
    db.collection("FAR").document(farmerId).collection("NOTIFICATION")
            .add(notificationData)
            .addOnSuccessListener(aVoid -> Log.d("Notification", "Notification sent successfully."))
            .addOnFailureListener(e -> Log.e("Notification", "Error sending notification.", e));
}

```

# PostControlActivity

## Overview
This activity handles the management of posts by the user.

## Methods

### `onCreate(Bundle savedInstanceState)`
- Initializes the activity.
- Sets up click listeners for buttons.
- Loads the user's posts.

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_post_control);

    btnCreatePost=findViewById(R.id.btnCreatePost);
    btnCreatePost.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Intent intent = new Intent(PostControlActivity.this, CreatePostActivity.class);
            intent.putExtra("UserID",userID);
            startActivity(intent);
        }
    });

    imgHome = findViewById(R.id.imgHome);
    imgProfile = findViewById(R.id.imgFarmerProfile);

    userID = getUserID();
    if (userID != null && !userID.isEmpty()) {
        Toast.makeText(this, "UserID: " + userID, Toast.LENGTH_SHORT).show();
    } else {
        Toast.makeText(this, "Error: User ID is missing.", Toast.LENGTH_SHORT).show();
    }

    swipeRefreshLayout = findViewById(R.id.swipeRefreshLayoutPostControl);
    swipeRefreshLayout.setOnRefreshListener(this::loadPosts);

    loadPosts();

    imgHome.setOnClickListener(v -> navigateToFarmerDashboardActivity());

    imgProfile.setOnClickListener(v -> navigateToEditProfileActivity(userID));

}
```

### `loadPosts()`
- Retrieves and displays the user's posts from the Firestore database.
- Handles the UI layout for each post, including buttons for editing and deleting posts.
```
private void loadPosts() {
    String userID = getUserID();
    if (userID == null || userID.trim().isEmpty()) {
        Toast.makeText(this, "Error: User ID is missing.", Toast.LENGTH_SHORT).show();
        Log.e("PostControlActivity", "Error: User ID is missing.");
        return;
    }
    FirebaseFirestore db = FirebaseFirestore.getInstance();

    db.collection("FAR").document(userID).collection("POST")
            .get()
            .addOnSuccessListener(queryDocumentSnapshots -> {
                LinearLayout postsLayout =   findViewById(R.id.layoutPostsPostControl);
                postsLayout.removeAllViews();
                for (QueryDocumentSnapshot document : queryDocumentSnapshots) {
                    View postView = LayoutInflater.from(PostControlActivity.this).inflate(R.layout.self_item_post, postsLayout, false);

                    TextView tvPostName = postView.findViewById(R.id.tvPostName);
                    TextView tvPostProduct = postView.findViewById(R.id.tvPostProduct);
                    TextView tvPostPrice = postView.findViewById(R.id.tvPostPrice);
                    TextView tvPostQuantity = postView.findViewById(R.id.tvPostQuantity);
                    TextView tvPostReleaseDate = postView.findViewById(R.id.tvPostReleaseDate);
                    TextView tvPostNote = postView.findViewById(R.id.tvPostNote);
                    ImageView imgPostImage = postView.findViewById(R.id.imgPostImage);
                    Button btnEditPost = postView.findViewById(R.id.btnEditPost);
                    Button btnDeletePost = postView.findViewById(R.id.btnDeletePost);

                    tvPostName.setText(document.getString("name"));
                    tvPostProduct.setText(document.getString("product"));
                    tvPostPrice.setText(document.getString("price"));
                    tvPostQuantity.setText(document.getString("quantity"));
                    tvPostReleaseDate.setText(document.getString("releaseDate"));
                    tvPostNote.setText(document.getString("note"));
                    Glide.with(PostControlActivity.this).load(document.getString("imageUrl")).into(imgPostImage);

                    btnEditPost.setOnClickListener(v -> {
                        Intent editIntent = new Intent(PostControlActivity.this, EditPostActivity.class);
                        editIntent.putExtra("postID", document.getId());
                        editIntent.putExtra("userID", userID);
                        startActivity(editIntent);
                    });

                    btnDeletePost.setOnClickListener(v -> {
                        document.getReference().delete().addOnSuccessListener(aVoid -> {
                            Toast.makeText(PostControlActivity.this, "Post deleted successfully", Toast.LENGTH_SHORT).show();
                            loadPosts();
                        }).addOnFailureListener(e -> Toast.makeText(PostControlActivity.this, "Error deleting post", Toast.LENGTH_SHORT).show());
                    });

                    postsLayout.addView(postView);
                }
                swipeRefreshLayout.setRefreshing(false);
            })
            .addOnFailureListener(e -> {
                swipeRefreshLayout.setRefreshing(false);
                Toast.makeText(PostControlActivity.this, "Error loading posts: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            });
}

```


