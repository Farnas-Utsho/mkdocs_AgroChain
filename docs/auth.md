## User Authentication
***
User authentication in AgroChain allows users to register and log in to the app, selecting their role from options such as farmer, wholesaler, retailer, government authority, or customer. During registration, users provide essential details like name, age, phone number, district, and National ID (NID) number. Additionally, they choose the type of product they wish to sell and set up a password
### User Registration Process

Users can register in the AgroChain mobile app by following these steps:

1. Launch the App:
Open the AgroChain mobile app on your device.


2. Tap on "Register":
On the app's welcome screen, tap on the "Register" button to begin the registration process.
 ![](img\auth\1.jpg)
3. Select User Role:
Choose your role from the available options, such as farmer, wholesaler, retailer, government authority, or customer.
![](img\auth\3.jpg)

4. Provide Personal Details:
Fill in the required fields with your personal information, including:

    Name
    Age
    Phone number
    District
    National ID (NID) number
![](img\auth\4.jpg)
5. Choose Product Type:
Select the type of product you intend to sell or purchase from the dropdown menu provided.
![](img\auth\5.jpg)
6. Set Password:
Create a password for your account by entering it into the designated field. Confirm the password to ensure accuracy.

7. Confirm Registration:
Review the information you've provided and ensure it is accurate. Once satisfied, click the "Confirm Registration" button to complete the registration process.
![](img\auth\5.jpg)
8. Verification:
Depending on the user role selected, additional verification steps may be required. For example, farmers may need to verify their farm ownership, while government authorities may need to undergo an approval process.

9. Confirmation Email/SMS:
Upon successful registration, you may receive a confirmation email or SMS to verify your account. Follow the instructions provided to confirm your registration.

10. Login:
Once your registration is confirmed, you can log in to the AgroChain app using your registered credentials and begin exploring its features.
![](img\auth\6.jpg)

## Here's how the authentication process is implemented in code:

1.  The loginUser() method handles the login process by verifying the user's credentials against the Firestore database. Based on the user type, the app redirects the user to the appropriate dashboard upon successful login.
```
  // LoginActivity.java

// Method to handle user login
private void loginUser() {
    String userID = editTextUserID.getText().toString().trim();
    String password = editTextPassword.getText().toString().trim();

    // Extract prefix from user ID to determine user type
    String prefix = userID.split("-")[0];
    String collectionPath = mapPrefixToCollection(prefix);

    // Query Firestore to check user credentials
    db.collection(collectionPath).document(userID).get().addOnCompleteListener(task -> {
        if (task.isSuccessful()) {
            DocumentSnapshot document = task.getResult();
            if (document != null && document.exists()) {
                String storedPassword = document.getString("password");
                if (storedPassword != null && storedPassword.equals(password)) {
                    String userType = document.getString("userType");
                    saveUserIDToPrefs(userID);
                    redirectToDashboard(userType, userID);
                } else {
                    // Incorrect password
                    Toast.makeText(LoginActivity.this, "Incorrect password.", Toast.LENGTH_LONG).show();
                }
            } else {
                // User not found
                Toast.makeText(LoginActivity.this, "User not found.", Toast.LENGTH_LONG).show();
            }
        } else {
            // Error fetching user details
            Log.e("Firestore Error", "Error fetching user details", task.getException());
            Toast.makeText(LoginActivity.this, "Login failed. Please try again later.", Toast.LENGTH_LONG).show();
        }
    });
}

```

2. The mapPrefixToCollection() method maps the prefix of the user ID to the corresponding Firestore collection, based on the user's role.
```
    // LoginActivity.java

// Method to map user ID prefix to Firestore collection
private String mapPrefixToCollection(String prefix) {
    switch (prefix) {
        case "FAR":
            return "FAR";
        case "GOV":
            return "GOV";
        case "WHO":
            return "WHO";
        case "CUS":
            return "CUS";
        default:
            return "users";
    }
}


```
3.    The moveToConfirmRegistrationActivity method gathers user input from the registration form, creates a data bundle containing the user's details, and navigates the user to the confirmation registration activity for finalizing the registration process.
        
```
       // RegisterActivity.java

private void initializeViews() {
    // Initialize views and components
    // Set up click listener for the "Next" button
    buttonNext.setOnClickListener(v -> moveToConfirmRegistrationActivity());
}

private void moveToConfirmRegistrationActivity() {
    // Gather user data from input fields and spinners
    Map<String, Object> userData = new HashMap<>();
    userData.put("name", editTextName.getText().toString());
    userData.put("age", editTextAge.getText().toString());
    userData.put("gender", spinnerGender.getSelectedItem().toString());
    userData.put("division", spinnerDivision.getSelectedItem().toString());
    userData.put("district", spinnerDistrict.getSelectedItem().toString());
    userData.put("upazila", spinnerUpazila.getSelectedItem().toString());
    userData.put("phone", editTextPhone.getText().toString());
    userData.put("userType", spinnerUserType.getSelectedItem().toString());

    // Create an intent to move to the confirmation registration activity
    Intent intent = new Intent(RegisterActivity.this, ConfirmRegistrationActivity.class);
    // Pass user data as serializable extra with the intent
    intent.putExtra("userData", (Serializable) userData);
    // Start the confirmation registration activity
    startActivity(intent);
}

 ```



4. In saveUserDetails method, the user's details, including the profile picture URL, are saved to Firestore. The profile picture is uploaded to Firebase Storage, and upon successful upload, the URL is obtained and included in the user's data before saving it to Firestore..

```
// ConfirmRegistrationActivity.java

private void saveUserDetails(final String userId) {
    // Prepare Firestore collection path based on user type prefix
    String collectionPath = userTypePrefix;

    // Get a reference to Firebase Storage to store profile picture
    StorageReference storageRef = FirebaseStorage.getInstance().getReference("user_profile_images/" + userId + ".jpg");

    // Convert default profile picture to byte array for uploading
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.default_profile);
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, baos);
    byte[] data = baos.toByteArray();

    // Upload profile picture to Firebase Storage
    UploadTask uploadTask = storageRef.putBytes(data);
    uploadTask.addOnSuccessListener(taskSnapshot -> {
        taskSnapshot.getStorage().getDownloadUrl().addOnSuccessListener(uri -> {
            String imageUrl = uri.toString();
            userData.put("profilePictureUrl", imageUrl);

            // Set user data in Firestore document
            db.collection(collectionPath).document(userId)
                    .set(userData)
                    .addOnSuccessListener(aVoid -> Toast.makeText(ConfirmRegistrationActivity.this, "User Details Saved Successfully with Profile Image", Toast.LENGTH_SHORT).show())
                    .addOnFailureListener(e -> Toast.makeText(ConfirmRegistrationActivity.this, "Failed to Save User Details", Toast.LENGTH_SHORT).show());
        });
    }).addOnFailureListener(e -> {
        Toast.makeText(Confir

```

5. The redirectToDashboard() method redirects the user to the appropriate dashboard based on their user type. It also passes the user ID as an extra parameter to the dashboard activity.
```
    // LoginActivity.java

// Method to redirect user to appropriate dashboard
private void redirectToDashboard(String userType, String userID) {
    Intent intent=null;
    switch (userType) {
        case "Farmer":
            intent = new Intent(LoginActivity.this, FarmerDashboardActivity.class);
            break;
        case "Wholesaler":
            intent = new Intent(LoginActivity.this, WholesalerDashboardActivity.class);
            break;
        case "Government Authority":
            intent = new Intent(LoginActivity.this, GovernmentDashboardActivity.class);
            break;
        case "Retailer":
            intent = new Intent(LoginActivity.this, RetailerDashboardActivity.class);
            break;
        case "Customer":
            intent = new Intent(LoginActivity.this, CustomerDashboardActivity.class);
            break;
        default:
            return;
    }
    if (intent != null) {
        intent.putExtra("userID", userID);
        startActivity(intent);
    } else {
        Toast.makeText(LoginActivity.this, "Unknown user type: " + userType, Toast.LENGTH_LONG).show();
    }
    startActivity(intent);
}

```
