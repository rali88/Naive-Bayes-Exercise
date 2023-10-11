# System Design Document
## Fire Equipment Exchange Program Platform
### Functional Requirements

1. **User Management**

2. **Item Posting**

3. **Search and Browse**

4. **Buyer Communication**

5. **Community**

6. **Transactions**

7. **Administration**

8. **Mobile Experience**

### System Architecture
1. **Onion Architecture Model**
2. **Technologies Used**
   - ASP.NET Web Forms
   - C#
   - MS SQL Server
   - Azure PaaS

## 1. User Management

### 1.1 User Registration
User registration is a fundamental part of the Fire Equipment Exchange Program Platform. Here's how it's technically implemented:

- **ASP.NET Web Forms**: User registration is presented to users through ASP.NET web forms.
- **MS SQL Server Database**: User data is stored in an MS SQL Server database, ensuring data persistence and retrieval.

**Code Snippet: User Registration Form (ASP.NET Web Form)**
```html
<!-- Registration.aspx -->
<form id="registrationForm" runat="server">
    <div>
        <label for="username">User Name:</label>
        <asp:TextBox ID="username" runat="server"></asp:TextBox>
    </div>
    <div>
        <label for="deptName">Fire Department Name:</label>
        <asp:TextBox ID="deptName" runat="server"></asp:TextBox>
    </div>
    <!-- Other registration fields: Fire Department ID, email, phone, address -->
    <div>
        <asp:Button ID="registerButton" Text="Register" runat="server" OnClick="RegisterUser" />
    </div>
</form>
```

**C# Code for User Registration**
```csharp
// Registration.aspx.cs
protected void RegisterUser(object sender, EventArgs e)
{
    // Collect user input
    string userName = username.Text;
    string departmentName = deptName.Text;
    // Extract other user registration data

    // Store the user data in the database
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        string query = "INSERT INTO Users (UserName, DepartmentName, ...) VALUES (@UserName, @DepartmentName, ...)";
        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@UserName", userName);
            command.Parameters.AddWithValue("@DepartmentName", departmentName);
            // Add parameters for other registration fields
            command.ExecuteNonQuery();
        }
    }
    // Redirect to a confirmation page or perform other actions
}
```

### 1.2 User Roles
User roles define the permissions and access levels for different user types (Admins, Chiefs/Department Liaisons, Basic Users). These roles are managed through role-based access control (RBAC) using ASP.NET Identity.

**Code Snippet: Assigning User Roles (C#)**
```csharp
// Assign the "Admin" role to a user
string adminRole = "Admin";
string userId = "user123"; // The user's unique identifier
if (!Roles.IsUserInRole(userId, adminRole))
{
    Roles.AddUserToRole(userId, adminRole);
}
```

### 1.3 User Profile Management
Users can edit their profiles, including their phone number and address. This is implemented through ASP.NET web forms and database updates.

**Code Snippet: User Profile Edit (ASP.NET Web Form)**
```html
<!-- UserProfile.aspx -->
<form id="profileForm" runat="server">
    <div>
        <label for="phone">Phone:</label>
        <asp:TextBox ID="phone" runat="server"></asp:TextBox>
    </div>
    <div>
        <label for="address">Address:</label>
        <asp:TextBox ID="address" runat="server"></asp:TextBox>
    </div>
    <div>
        <asp:Button ID="updateButton" Text="Update Profile" runat="server" OnClick="UpdateProfile" />
    </div>
</form>
```

**C# Code for User Profile Update**
```csharp
// UserProfile.aspx.cs
protected void UpdateProfile(object sender, EventArgs e)
{
    // Collect user updates
    string newPhone = phone.Text;
    string newAddress = address.Text;

    // Update the user's profile in the database
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        string query = "UPDATE Users SET Phone = @Phone, Address = @Address WHERE UserId = @UserId";
        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@Phone", newPhone);
            command.Parameters.AddWithValue("@Address", newAddress);
            command.Parameters.AddWithValue("@UserId", userId); // Identify the user
            command.ExecuteNonQuery();
        }
    }
    // Redirect to a confirmation page or perform other actions
}
```

User management is a critical component of the platform, ensuring that users can access and utilize the features based on their roles and update their profile information as needed.

## 2. Item Posting

### 2.1 Equipment Listing Creation
Users can create new listings for equipment exchange with various details. This involves both capturing textual information and uploading media content. Here's how it's implemented:

- **ASP.NET Web Forms**: The equipment listing creation form is presented to users through ASP.NET web forms.
- **Media Upload**: Users can upload photos and videos to showcase the equipment.
- **Text Data Entry**: Users provide a title, description, condition, maintenance history, available dates, pickup location, pricing, and price notes.
- **Approval Process**: All equipment listings go through an approval process before being published.

**Code Snippet: Equipment Listing Creation Form (ASP.NET Web Form)**
```html
<!-- EquipmentListing.aspx -->
<form id="listingForm" runat="server">
    <div>
        <label for="title">Title:</label>
        <asp:TextBox ID="title" runat="server"></asp:TextBox>
    </div>
    <!-- Other equipment listing fields: description, condition, maintenance history, available dates, pickup location, pricing, price notes -->
    <div>
        <asp:FileUpload ID="mediaUpload" runat="server" />
    </div>
    <div>
        <asp:Button ID="createListingButton" Text="Create Listing" runat="server" OnClick="CreateListing" />
    </div>
</form>
```

**C# Code for Equipment Listing Creation**
```csharp
// EquipmentListing.aspx.cs
protected void CreateListing(object sender, EventArgs e)
{
    // Collect user input
    string listingTitle = title.Text;
    // Extract other equipment listing data

    // Upload and store media content
    if (mediaUpload.HasFile)
    {
        string mediaFileName = Path.GetFileName(mediaUpload.FileName);
        string mediaPath = Server.MapPath("~/Media/") + mediaFileName;
        mediaUpload.SaveAs(mediaPath);
    }

    // Store the equipment listing data in the database
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        string query = "INSERT INTO EquipmentListings (Title, ..., MediaPath) VALUES (@Title, ..., @MediaPath)";
        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@Title", listingTitle);
            // Add parameters for other equipment listing fields
            command.Parameters.AddWithValue("@MediaPath", mediaPath);
            command.ExecuteNonQuery();
        }
    }
    // Redirect to a confirmation page or perform other actions
}
```

### 2.2 Equipment Listing Approval Process
All equipment listings go through an approval process managed by administrators. Here's how the approval process is implemented:

- **Admin Notification**: Once a listing is posted, an admin is alerted through text, email, or in-system messaging.
- **Admin Review**: Admins review the posted listing and take one of three actions: Publish, Send back for clarifications, or Reject.
- **Database Updates**: Admin actions (Publish, Send back, Reject) are reflected in the database, controlling the visibility of listings.

**Code Snippet: Admin Review and Actions (C#)**
```csharp
// EquipmentListingApproval.aspx.cs
protected void AdminReviewListing(string listingId)
{
    // Fetch the listing from the database
    EquipmentListing listing = FetchListingFromDatabase(listingId);

    // Admin review and action
    if (/* admin approves the listing */) {
        listing.Status = ListingStatus.Published;
    } else if (/* admin sends back for clarifications */) {
        listing.Status = ListingStatus.RequiresClarifications;
    } else if (/* admin rejects the listing */) {
        listing.Status = ListingStatus.Rejected;
    }

    // Update the listing's status in the database
    UpdateListingStatusInDatabase(listing);
}
```

## 3. Item Requests

### 3.1 Creating Item Requests
The solution enables users to create item requests when they need specific equipment or items. This feature ensures that potential providers can identify and respond to these requests. The creation and management of item requests are as follows:

- **ASP.NET Web Forms**: Users utilize ASP.NET web forms to create item requests, providing details such as the item name, keyword descriptions, location, and condition description.
- **Admin Approval Process**: Just like equipment listings, item requests are subject to an admin approval process before becoming visible on the platform.
- **Editing and Management**: Users have the capability to edit and manage their item requests. However, any edits trigger a re-evaluation through the Approval Process.
- **Matching Notifications**: When there are matching equipment listings available for a user's item request, the solution notifies the user. The item requests display location and contact information for communication with potential equipment providers.

**Code Snippet: Item Request Creation Form (ASP.NET Web Form)**
```html
<!-- ItemRequest.aspx -->
<form id="itemRequestForm" runat="server">
    <div>
        <label for="itemName">Item Name:</label>
        <asp:TextBox ID="itemName" runat="server"></asp:TextBox>
    </div>
    <!-- Other item request fields: keyword descriptions, location, condition description -->
    <div>
        <asp:Button ID="createItemRequestButton" Text="Create Item Request" runat="server" OnClick="CreateItemRequest" />
    </div>
</form>
```

**C# Code for Item Request Creation**
```csharp
// ItemRequest.aspx.cs
protected void CreateItemRequest(object sender, EventArgs e)
{
    // Collect user input
    string requestItemName = itemName.Text;
    // Extract other item request data

    // Store the item request data in the database
    using (SqlConnection connection = new SqlConnection(connectionString))
    {
        connection.Open();
        string query = "INSERT INTO ItemRequests (ItemName, ..., Status) VALUES (@ItemName, ..., @Status)";
        using (SqlCommand command = new SqlCommand(query, connection))
        {
            command.Parameters.AddWithValue("@ItemName", requestItemName);
            // Add parameters for other item request fields
            command.Parameters.AddWithValue("@Status", ItemRequestStatus.Pending);
            command.ExecuteNonQuery();
        }
    }
    // Redirect to a confirmation page or perform other actions
}
```

### 3.2 Matching Notifications
When there are matching equipment listings available for a user's item request, the solution notifies the user. This matching is based on various criteria, such as keywords, equipment type, and specifications. The notification mechanism can be implemented through email, text, or in-app notifications.

**Matching Notification Trigger (C#)**
```csharp
public void NotifyMatchingListings(ItemRequest request)
{
    // Query the database to find matching equipment listings
    List<EquipmentListing> matchingListings = FindMatchingListings(request);
    
    // Notify the user
    foreach (var listing in matchingListings)
    {
        // Send email, text, or in-app notification to the user
        SendMatchingNotification(request.UserId, listing);
    }
}
```

The item request feature improves user engagement by helping them find the equipment they need more effectively.

## 4. Search and Browse

### 4.1 Browsing

Users of the Fire Equipment Exchange Program Platform can have an intuitive browsing experience to discover equipment listings. This feature allows users to view a list of available equipment quickly. The browsing functionality is designed to be user-friendly and efficient.

- **ASP.NET Web Forms**: The browsing functionality is implemented using ASP.NET web forms, which display listings based on title and description keywords.

### 4.2 Search

In addition to browsing, users can search for equipment listings to find specific items they need. The search functionality enables users to enter keywords, equipment types, specifications, and other relevant details to filter the listings.

- **Search Engine**: The solution incorporates a search engine that queries the database to retrieve equipment listings based on user input.

- **Sorting Options**: Search results are sortable by relevance, date posted, and price, providing users with flexibility in finding the equipment that best matches their needs.

**Code Snippet: Equipment Listing Search (ASP.NET Web Form)**
```html
<!-- Search.aspx -->
<form id="searchForm" runat="server">
    <div>
        <label for="searchKeywords">Search Keywords:</label>
        <asp:TextBox ID="searchKeywords" runat="server"></asp:TextBox>
    </div>
    <!-- Other search criteria fields: equipment type, specifications, etc. -->
    <div>
        <asp:Button ID="searchButton" Text="Search" runat="server" OnClick="SearchEquipment" />
    </div>
</form>
```

**C# Code for Equipment Listing Search**
```csharp
// Search.aspx.cs
protected void SearchEquipment(object sender, EventArgs e)
{
    // Retrieve user search input
    string keywords = searchKeywords.Text;
    // Extract other search criteria

    // Query the database for matching equipment listings
    List<EquipmentListing> searchResults = SearchListings(keywords, /* other criteria */);

    // Display the search results
    DisplaySearchResults(searchResults);
}
```

The search and browse features are essential for helping users find the equipment they need efficiently.

## 5. Buyer Communication

### 5.1 Featured Item

The platform supports featuring specific items to promote them to potential buyers. Featured items are displayed at the top of the browser upon user login. Admins have the authority to select and feature items to attract more attention.

- **Admin Functionality**: Admins can select items to be featured through the platform's admin interface.

- **User Experience**: Users see featured items prominently upon logging in, increasing their visibility.

**Code Snippet: Admin Interface for Featuring Items (ASP.NET Web Form)**
```html
<!-- AdminInterface.aspx -->
<form id="featureItemsForm" runat="server">
    <div>
        <asp:ListBox ID="featureItemList" runat="server" SelectionMode="Multiple"></asp:ListBox>
    </div>
    <div>
        <asp:Button ID="featureItemsButton" Text="Feature Items" runat="server" OnClick="FeatureItems" />
    </div>
</form>
```

**C# Code for Admin Feature Items Functionality**
```csharp
// AdminInterface.aspx.cs
protected void FeatureItems(object sender, EventArgs e)
{
    // Retrieve the selected items to feature
    List<int> selectedListingIds = GetSelectedListingIds();
    
    // Feature the selected items in the database
    FeatureListings(selectedListingIds);
}
```

### 5.2 Messaging

The platform facilitates communication between buyers and sellers through a built-in messaging system. Users can initiate and participate in message threads with file attachments and images. While this feature is not required, it enhances user engagement and simplifies the negotiation process.

- **Messaging System**: The messaging system is implemented to allow users to send and receive messages within the platform.

- **File Attachments and Images**: Users can attach files and images to their messages, making it easier to share information about equipment listings.

## 5.3 Email Functionality

### Emailing Equipment Listings

Users can utilize the platform to email equipment listings to other potential buyers who are registered on the platform. This feature enhances sharing and collaboration among users.

- **Email Integration**: The platform integrates with email services to allow users to send equipment listings via email.

**Code Snippet: Email Equipment Listing (ASP.NET Web Form)**
```html
<!-- ListingDetails.aspx -->
<form id="emailListingForm" runat="server">
    <div>
        <label for="recipientEmail">Recipient's Email:</label>
        <asp:TextBox ID="recipientEmail" runat="server"></asp:TextBox>
    </div>
    <!-- Additional email details: subject, message, etc. -->
    <div>
        <asp:Button ID="emailListingButton" Text="Email Listing" runat="server" OnClick="EmailListing" />
    </div>
</form>
```

**C# Code for Emailing Equipment Listing**
```csharp
// ListingDetails.aspx.cs
protected void EmailListing(object sender, EventArgs e)
{
    // Retrieve recipient's email and other email details
    string recipientEmail = recipientEmail.Text;
    // Extract email subject, message, and listing details
    
    // Send the email using the platform's email integration
    SendEmail(recipientEmail, /* other email details */);
}
```

The email functionality enhances the platform's user experience by allowing users to easily share equipment listings with potential buyers.

## 5.4 Notifications

Users receive notifications through multiple channels when certain events occur on the platform. These notifications are sent via email, text messages, or in-app messages.

- **Notification Types**: Notifications are sent for events like inquiries from potential buyers, changes in listing status, and more.

- **Opt-in for SMS**: Users have the option to receive SMS notifications, allowing for real-time updates on platform activities.

## 6. Community

The community aspect of the platform promotes engagement and networking among users.

### 6.1 Public User Profiles

User profiles are designed to display essential information about fire departments and their personnel, including names, locations, and hours of operation. This feature adds transparency to the platform and helps users connect more effectively.

- **Profile Information**: User profiles are populated with data provided during registration, ensuring that fire departments' details are available to the community.

### 6.2 Sharing and Bookmarking

Users are equipped with tools for sharing and bookmarking equipment listings. These functions enable users to spread the word about equipment listings they find interesting and to save items for later reference.

- **Sharing**: Users can share equipment listings via email and other communication channels, expanding the reach of listings.

- **Bookmarking**: Users can save equipment listings to their personal collections for easy access.

In the upcoming sections, we will cover transaction-related functionalities and the administration aspects of the platform.

## 7. Transactions

### 7.1 Cart and Checkout Process

The platform facilitates transactions by allowing buyers to add equipment listings to their cart and complete the checkout process. This process is outlined as follows:

1. **Adding to Cart**: Buyers can add items to their cart, indicating their interest in purchasing those items.

2. **Seller Notification**: Sellers are notified via text, email, or platform message when a buyer adds their item to a cart.

3. **Negotiation**: Sellers and buyers negotiate the terms of the sale outside of the platform.

4. **Item Removal**: Sellers are responsible for removing sold items from the platform when a sale is confirmed.

5. **Confirmation or Re-Listing**: Once an item is placed into a cart, sellers receive notifications to either confirm the sale or re-list the item if the transaction was not successful.

- **ASP.NET Web Forms**: The cart and checkout process is implemented using ASP.NET web forms, ensuring a smooth user experience.

**Code Snippet: Cart Management (ASP.NET Web Form)**
```html
<!-- Cart.aspx -->
<form id="cartForm" runat="server">
    <!-- Cart management interface: add/remove items -->
    <div>
        <asp:Button ID="checkoutButton" Text="Proceed to Checkout" runat="server" OnClick="ProceedToCheckout" />
    </div>
</form>
```

**C# Code for Cart Management**
```csharp
// Cart.aspx.cs
protected void ProceedToCheckout(object sender, EventArgs e)
{
    // Implement the cart and checkout logic
}
```

## 8. Administration

The administration section of the platform is crucial for managing users, equipment listings, and item requests. Admins have the authority to approve, block, delete accounts, review and approve equipment listings and item requests, and access reports on platform performance. This section ensures the platform's integrity and effectiveness.

### 8.1 User Management

Admins can manage user accounts by performing actions such as approving, blocking, and deleting accounts. This authority helps maintain a safe and compliant user base.

### 8.2 Approval Process

The platform employs an approval process for equipment listings and item requests. This workflow involves two steps:

1. **Admin Alert**: When a listing is posted, an admin is alerted through text, email, or in-system messaging.

2. **Admin Review**: The admin reviews the posted listing and takes one of three actions: publishing, sending it back to the lister for clarifications, or rejecting the listing.

### 8.3 Report Generation

Admins can generate reports on platform performance, including metrics like the number of users, items added, items "sold/removed," and traffic/activity metrics. This reporting functionality provides insights into the platform's usage and effectiveness.

The administration section ensures that the platform operates smoothly and according to the guidelines and regulations set by the fire departments.

In the following sections, we will discuss the mobile experience and its relevant features.

## 9. Mobile Experience

The platform extends its capabilities to mobile devices, providing both iOS and Android applications with the same functionalities available on the website.

### 9.1 Push Notifications

Mobile users receive push notifications for various events, such as new messages, purchases, and other relevant updates. Users can opt-in for SMS notifications, allowing them to stay informed in real-time.

### 9.2 Mobile Listings

Mobile users can capture photos and directly list equipment from their mobile devices. This feature simplifies the process of posting new equipment listings.

### 9.3 Location-Aware Features (Nice to Have)

While not a required feature, location-aware functionalities provide information about nearby equipment listings and pickup points. Users can benefit from this feature when they need equipment urgently or prefer local options.

In the subsequent sections, we will provide an overview of the system architecture and design based on the Onion Architecture model and the technologies used.

## 10. System Architecture and Design

The Fire Equipment Exchange Program Platform is designed based on the Onion Architecture model, utilizing ASP.NET web forms, C#, and MS SQL Server databases. The application is hosted on Azure PaaS, ensuring scalability, availability, and efficient performance.

### 10.1 Onion Architecture Model

Onion Architecture is a software architectural pattern that separates an application into concentric circles or layers. It promotes a clear separation of concerns, making the application more maintainable and testable. The following layers are part of the Onion Architecture model for this platform:

1. **Core**: The innermost layer contains business logic and entities. It defines the fundamental application behavior, including user management, item posting, and buyer communication.

2. **Application**: The application layer holds application-specific logic, such as user interface components, services, and controllers for features like item requests and search functionality.

3. **Infrastructure**: The infrastructure layer handles external concerns, including database access and communication with external services. It contains repositories for data storage and email services for buyer communication.

4. **Presentation**: The presentation layer includes the user interface components, such as ASP.NET web forms, responsible for rendering the platform's web pages.

This architectural approach ensures that the core business logic remains independent of the user interface and infrastructure concerns.

### 10.2 Technologies Used

The Fire Equipment Exchange Program Platform leverages several key technologies:

- **ASP.NET Web Forms**: The primary technology for creating the web-based user interface, implementing the presentation layer of the Onion Architecture.

- **C#**: The programming language used throughout the application to develop business logic, controllers, and services.

- **MS SQL Server**: The relational database management system used to store user data, equipment listings, item requests, and other application data.

- **Azure PaaS**: The platform as a service (PaaS) offering by Microsoft Azure hosts the application, providing scalability, availability, and a range of cloud services to support the platform's operation.

- **iOS and Android**: Mobile applications for iOS and Android are developed to extend the platform's capabilities to mobile users.

This robust technology stack ensures a reliable and efficient platform that meets the functional requirements while maintaining a high level of scalability and flexibility.

## Conclusion

This System Design Document provides a comprehensive understanding of the system architecture and design for the Fire Equipment Exchange Program Platform. The platform caters to the specific needs of fire departments and authorized personnel, enabling them to manage equipment listings, item requests, and user profiles effectively.

The use of the Onion Architecture model, along with ASP.NET web forms, C#, MS SQL Server databases, and Azure PaaS, ensures a robust and scalable solution. The platform's features, including user management, item posting, buyer communication, community engagement, transactions, administration, and mobile experience, are designed to provide a seamless and efficient user experience.

This document serves as a roadmap for the development and deployment of the Fire Equipment Exchange Program Platform, ensuring that it meets the functional requirements while adhering to a well-structured and maintainable architectural model.
