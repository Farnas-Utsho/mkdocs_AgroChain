## Benefits of the user profile feature:

 ![](img\profile\11.jpg)

###    Personalization: Users can customize their profiles with details such as name, age, contact information, and location, tailoring their experience to their specific needs.


1. Update Profile Information: Users can modify and update their profile details such as name, age, contact information, and address, ensuring that their profile remains accurate and up-to-date to reflect any changes in their personal information.

```
    private void updateUserProfile(String imageUrl) {
    Map<String, Object> userUpdates = new HashMap<>();
    userUpdates.put("name", editTextName.getText().toString());
    userUpdates.put("age", editTextAge.getText().toString());
    userUpdates.put("phone", editTextPhone.getText().toString());
    userUpdates.put("gender", spinnerGender.getSelectedItem().toString());
    userUpdates.put("division", editTextDivision.getText().toString());
    userUpdates.put("district", editTextDistrict.getText().toString());
    userUpdates.put("upazila", editTextUpazila.getText().toString());
    userUpdates.put("NID", editTextNID.getText().toString());
    if (imageUrl != null) {
        userUpdates.put("profilePictureUrl", imageUrl);
    }

    db.collection("FAR").document(userID).update(userUpdates)
            .addOnSuccessListener(aVoid -> {
                Toast.makeText(EditProfileActivity.this, "Profile Updated Successfully", Toast.LENGTH_SHORT).show();
            })
            .addOnFailureListener(e -> Toast.makeText(EditProfileActivity.this, "Failed to update profile", Toast.LENGTH_SHORT).show());
}


```


2. Delete Profile Picture: Users have the option to remove or delete their existing profile picture, providing them with flexibility in managing their visual representation within the app based on their preferences or changing circumstances.
```
private void deleteUserProfile() {
        db.collection("FAR").document(userID).delete().addOnSuccessListener(aVoid -> {
            Toast.makeText(EditProfileActivity.this, "Profile Deleted Successfully", Toast.LENGTH_SHORT).show();
            startActivity(new Intent(EditProfileActivity.this, FarmerDashboardActivity.class));
            finish();
        }).addOnFailureListener(e -> Toast.makeText(EditProfileActivity.this, "Failed to delete profile", Toast.LENGTH_SHORT).show());
    }

```

3. Upload Profile Picture: Through a simple interface, users can upload images from their device's gallery or camera directly to their profile, enabling them to easily update their profile picture with their preferred image or photo.
```
private void uploadNewProfilePicture() {
        if (selectedImageUri != null) {
            String fileName = "profile_" + userID + ".jpg";
            StorageReference fileRef = FirebaseStorage.getInstance().getReference().child("profile_images/" + fileName);

            Log.d("EditProfileActivity", "Uploading new profile picture: " + selectedImageUri.toString());

            fileRef.putFile(selectedImageUri).addOnSuccessListener(taskSnapshot -> {
                fileRef.getDownloadUrl().addOnSuccessListener(uri -> {
                    String imageUrl = uri.toString();
                    Log.d("EditProfileActivity", "New profile picture uploaded. Image URL: " + imageUrl);
                    updateUserProfile(imageUrl);
                });
            }).addOnFailureListener(e -> {
                Log.e("EditProfileActivity", "Upload failed: " + e.getMessage(), e);
                Toast.makeText(EditProfileActivity.this, "Upload failed: " + e.getMessage(), Toast.LENGTH_SHORT).show();
            });
        } else {
            Log.d("EditProfileActivity", "No new profile picture selected. Updating profile without changing the image.");
            updateUserProfile(null);
        }
    }

```