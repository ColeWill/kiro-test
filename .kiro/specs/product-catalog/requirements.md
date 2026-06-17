# Requirements Document

## Introduction

A React-based product catalog feature that fetches and displays products from the DummyJSON API. Users can browse, search, filter, and sort a collection of 20 products. The feature uses Redux Toolkit for state management, TypeScript for type safety, and follows a feature-based folder structure with custom hooks for logic separation.

## Glossary

- **Product_Catalog**: The React feature module responsible for fetching, displaying, filtering, sorting, and searching products
- **Product**: A data entity containing a title, price, category, and rating fetched from the DummyJSON API
- **Product_List**: The UI component that renders the collection of Product items
- **Category_Filter**: The UI control that allows users to filter products by one or more categories
- **Price_Range_Filter**: The UI control that allows users to filter products by minimum and maximum price values
- **Sort_Control**: The UI control that allows users to order products by price or rating
- **Search_Input**: The text input field where users type a product name query
- **Result_Counter**: The UI element that displays the count of products currently visible after all filters and search are applied
- **DummyJSON_API**: The external REST API at dummyjson.com/products used as the data source
- **Product_Catalog_Slice**: The Redux Toolkit slice managing the product catalog state

## Requirements

### Requirement 1: Fetch Products from API

**User Story:** As a user, I want the application to load products automatically, so that I can browse available items without manual action.

#### Acceptance Criteria

1. WHEN the Product_Catalog mounts, THE Product_Catalog SHALL fetch 20 products from the DummyJSON_API endpoint `https://dummyjson.com/products?limit=20`
2. WHILE the Product_Catalog is fetching data, THE Product_Catalog SHALL display a loading indicator, and SHALL remove the loading indicator within 1 second of receiving a successful response or an error
3. WHEN the DummyJSON_API returns a successful response, THE Product_Catalog_Slice SHALL store the products containing title, price, category, and rating fields
4. IF the DummyJSON_API returns an error, the request fails, or no response is received within 10 seconds, THEN THE Product_Catalog SHALL display an error message indicating the nature of the failure
5. IF the Product_Catalog unmounts and remounts while a fetch is in progress, THEN THE Product_Catalog SHALL cancel the pending request and initiate a new fetch

### Requirement 2: Display Product List

**User Story:** As a user, I want to see products displayed in a list, so that I can browse what is available.

#### Acceptance Criteria

1. WHEN products are loaded, THE Product_List SHALL render each Product displaying its title, price, category, and rating
2. THE Product_List SHALL format the price as a currency value with a dollar sign and two decimal places (e.g., "$9.99")
3. THE Product_List SHALL display the rating as a numeric value rounded to one decimal place followed by "/5" (e.g., "4.3/5")
4. IF the product list is empty after loading completes, THEN THE Product_List SHALL display a message indicating that no products are available

### Requirement 3: Filter by Category

**User Story:** As a user, I want to filter products by category, so that I can narrow down products to a specific type.

#### Acceptance Criteria

1. WHEN products are loaded, THE Category_Filter SHALL display all unique categories derived from the fetched products in alphabetical order
2. WHEN the user selects or deselects a category in the Category_Filter, THE Product_List SHALL immediately display only products whose category matches one of the currently selected categories without requiring a submit action
3. WHEN no categories are selected in the Category_Filter, THE Product_List SHALL display all products (subject to other active filters)

### Requirement 4: Filter by Price Range

**User Story:** As a user, I want to filter products by price range, so that I can find products within my budget.

#### Acceptance Criteria

1. WHEN the user sets a minimum price in the Price_Range_Filter, THE Product_List SHALL display only products with a price greater than or equal to the minimum value, where the minimum value is constrained to 0 or greater
2. WHEN the user sets a maximum price in the Price_Range_Filter, THE Product_List SHALL display only products with a price less than or equal to the maximum value
3. WHEN both minimum and maximum prices are set, THE Product_List SHALL display only products with a price within the inclusive range
4. WHEN products are loaded, THE Price_Range_Filter SHALL default to the full price range of the fetched products, with the minimum input set to the lowest product price and the maximum input set to the highest product price
5. IF the user sets a minimum price greater than the maximum price, THEN THE Price_Range_Filter SHALL prevent the submission and indicate that the minimum must be less than or equal to the maximum
6. THE Price_Range_Filter SHALL accept only numeric values with up to two decimal places and reject non-numeric input

### Requirement 5: Sort Products

**User Story:** As a user, I want to sort products by price or rating, so that I can find the best deals or highest-rated items.

#### Acceptance Criteria

1. THE Sort_Control SHALL provide options to sort by price ascending, price descending, rating ascending, rating descending, and a default option that applies no sorting
2. WHEN the user selects a sort option, THE Product_List SHALL reorder the displayed products according to the selected criterion and direction, preserving the original API order for products with equal values in the sorted field
3. WHEN no sort option is selected or the user selects the default option, THE Product_List SHALL display products in the order received from the DummyJSON_API

### Requirement 6: Search by Product Name

**User Story:** As a user, I want to search for products by name, so that I can quickly find a specific item.

#### Acceptance Criteria

1. WHEN the user types text into the Search_Input, THE Product_List SHALL display only products whose title contains the search text (case-insensitive substring match)
2. WHEN the Search_Input is cleared or contains only whitespace characters, THE Product_List SHALL display all products (subject to other active filters)
3. WHEN the user stops typing in the Search_Input, THE Product_List SHALL update the filtered results within 300 milliseconds of the last keystroke without requiring a submit action
4. IF the search text matches no product titles, THEN THE Product_List SHALL display an empty state message indicating that no products match the current search text

### Requirement 7: Display Filtered Result Count

**User Story:** As a user, I want to see how many products match my current filters, so that I know the scope of my search results.

#### Acceptance Criteria

1. THE Result_Counter SHALL display the number of products currently visible in the Product_List in the format "N product(s) found" where N is the count of displayed products
2. WHEN filters, search, or sort criteria change, THE Result_Counter SHALL update synchronously with the Product_List re-render to reflect the new count
3. IF the count is zero, THEN THE Result_Counter SHALL display "0 products found"

### Requirement 8: Combine Filters

**User Story:** As a user, I want all filters, search, and sort to work together, so that I can refine my product view with multiple criteria simultaneously.

#### Acceptance Criteria

1. WHEN multiple filter criteria are active (one or more categories selected in the Category_Filter, a minimum or maximum price set in the Price_Range_Filter, or text entered in the Search_Input), THE Product_Catalog SHALL apply all active filters using AND logic to determine the displayed products
2. THE Product_Catalog SHALL apply sorting after filtering, so the sort order applies only to the filtered subset
3. IF the combination of active filters results in zero matching products, THEN THE Product_Catalog SHALL display an empty state message indicating that no products match the current filter criteria

### Requirement 9: Responsive Layout

**User Story:** As a user, I want the product catalog to be usable on different screen sizes, so that I can browse products on mobile, tablet, and desktop devices.

#### Acceptance Criteria

1. WHILE the viewport width is 768 pixels or greater, THE Product_Catalog SHALL display the filter controls alongside the Product_List in a multi-column layout
2. WHILE the viewport width is less than 768 pixels, THE Product_Catalog SHALL stack the filter controls above the Product_List in a single-column layout
3. WHILE the viewport width is less than 768 pixels, THE Product_List SHALL display 1 product item per row
4. WHILE the viewport width is at least 768 pixels and less than 1024 pixels, THE Product_List SHALL display 2 product items per row
5. WHILE the viewport width is 1024 pixels or greater, THE Product_List SHALL display 3 product items per row
