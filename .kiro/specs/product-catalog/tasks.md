# Implementation Plan: Product Catalog

## Overview

Build a React product catalog feature using TypeScript and Redux Toolkit. The feature fetches 20 products from the DummyJSON API and provides client-side filtering (category, price range), sorting (price, rating), search by name, result counting, and responsive layout. All logic is encapsulated in custom hooks, with pure functions for filtering/sorting tested via property-based tests.

## Tasks

- [ ] 1. Set up project structure, types, and shared hooks
  - [ ] 1.1 Create feature directory and TypeScript types
    - Create `src/features/product-catalog/productCatalog.types.ts` with `Product`, `ApiProduct`, `SortField`, `SortDirection`, `SortOption`, `SortOptionKey`, and `ProductCatalogState` interfaces/types
    - Create `src/features/product-catalog/index.ts` barrel export file
    - _Requirements: 1.3, 2.1_

  - [ ] 1.2 Create shared utility hooks
    - Create `src/shared/hooks/useDebounce.ts` — generic debounce hook that delays value updates by a specified number of milliseconds
    - Create `src/shared/hooks/useMediaQuery.ts` — hook that returns a boolean for whether a CSS media query matches, using `window.matchMedia`
    - _Requirements: 6.3, 9.1, 9.2_

- [ ] 2. Implement Redux slice and service layer
  - [ ] 2.1 Create API service
    - Create `src/features/product-catalog/productCatalogService.ts`
    - Implement `fetchProductsFromApi` function that calls `https://dummyjson.com/products?limit=20` with an optional `AbortSignal`
    - Map API response to trimmed `Product` objects (id, title, price, category, rating)
    - Implement 10-second timeout via `AbortController`
    - _Requirements: 1.1, 1.3, 1.4, 1.5_

  - [ ] 2.2 Create Redux slice with async thunk and selectors
    - Create `src/features/product-catalog/productCatalogSlice.ts`
    - Define initial state matching `ProductCatalogState` interface
    - Implement `fetchProducts` async thunk with abort support
    - Implement reducers: `setSelectedCategories`, `setPriceRange`, `setSortOption`, `setSearchQuery`
    - Implement memoized selector `selectFilteredAndSortedProducts` that applies category filter → price range filter → search filter → sort in sequence
    - Implement helper selectors: `selectCategories` (unique, alphabetically sorted), `selectPriceRangeBounds` (min/max from data)
    - _Requirements: 1.1, 1.3, 1.4, 1.5, 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 5.1, 5.2, 5.3, 6.1, 6.2, 8.1, 8.2_

  - [ ] 2.3 Register slice in Redux store
    - Update `src/app/store.ts` to add the `productCatalog` reducer
    - Export `RootState` and `AppDispatch` types
    - _Requirements: 1.3_

  - [ ]\* 2.4 Write property tests for product mapping (Property 1)
    - **Property 1: Product mapping preserves required fields**
    - Generate arbitrary API product objects and verify mapping produces Product with identical id, title, price, category, rating
    - **Validates: Requirements 1.3**

  - [ ]\* 2.5 Write property tests for filtering and sorting logic (Properties 4–9, 11)
    - **Property 4: Category extraction is unique and alphabetically sorted**
    - **Property 5: Category filter returns exactly matching products**
    - **Property 6: Price range filter returns exactly products within bounds**
    - **Property 7: Default price range equals data bounds**
    - **Property 8: Sort produces correctly ordered output with stable tie-breaking**
    - **Property 9: Search filter returns exactly title-matching products**
    - **Property 11: Combined filters apply AND logic**
    - Create `src/features/product-catalog/productCatalog.property.test.ts` with all property tests using fast-check (min 100 iterations each)
    - **Validates: Requirements 3.1, 3.2, 4.1, 4.2, 4.3, 4.4, 5.2, 6.1, 8.1**

- [ ] 3. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Implement custom hooks
  - [ ] 4.1 Create useProductCatalog hook
    - Create `src/features/product-catalog/useProductCatalog.ts`
    - Dispatch `fetchProducts` on mount with abort cleanup on unmount
    - Return `products`, `filteredProducts`, `isLoading`, `error`, and `columns` (computed from `useMediaQuery`)
    - Use breakpoints: <768px → 1 column, 768–1023px → 2 columns, ≥1024px → 3 columns
    - _Requirements: 1.1, 1.2, 1.4, 1.5, 9.3, 9.4, 9.5_

  - [ ] 4.2 Create useProductFilters hook
    - Create `src/features/product-catalog/useProductFilters.ts`
    - Read filter state from Redux store via selectors
    - Provide handler functions for category change, price range change (with validation), sort change, and search change (with debounce)
    - Validate price inputs: reject non-numeric, constrain min ≥ 0, detect min > max and surface error
    - Use `useDebounce` for search query with 300ms delay
    - _Requirements: 3.2, 4.1, 4.2, 4.5, 4.6, 5.1, 6.1, 6.3_

- [ ] 5. Implement UI components
  - [ ] 5.1 Create ProductCard component
    - Create `src/features/product-catalog/ProductCard.tsx`
    - Display product title, formatted price (`$X.XX`), category, and formatted rating (`X.X/5`)
    - Implement price formatting utility and rating formatting utility
    - _Requirements: 2.1, 2.2, 2.3_

  - [ ]\* 5.2 Write property tests for formatting utilities (Properties 2, 3, 10)
    - **Property 2: Price formatting produces correct currency string**
    - **Property 3: Rating formatting produces correct display string**
    - **Property 10: Result counter displays correct count format**
    - **Validates: Requirements 2.2, 2.3, 7.1**

  - [ ] 5.3 Create ProductList component
    - Create `src/features/product-catalog/ProductList.tsx`
    - Render a responsive grid of `ProductCard` components using the `columns` prop
    - Display empty state message when product list is empty after loading
    - Display "No products match your current filters" when filters yield zero results
    - _Requirements: 2.1, 2.4, 6.4, 8.3_

  - [ ] 5.4 Create FilterPanel and filter components
    - Create `src/features/product-catalog/FilterPanel.tsx` — container using `useProductFilters` hook
    - Create `src/features/product-catalog/CategoryFilter.tsx` — checkbox group for categories
    - Create `src/features/product-catalog/PriceRangeFilter.tsx` — min/max number inputs with validation error display
    - Create `src/features/product-catalog/SortControl.tsx` — dropdown with 5 sort options (default, price asc/desc, rating asc/desc)
    - Create `src/features/product-catalog/SearchInput.tsx` — text input bound to search state
    - _Requirements: 3.1, 3.2, 3.3, 4.1, 4.2, 4.4, 4.5, 4.6, 5.1, 6.1, 6.2, 6.3_

  - [ ] 5.5 Create ResultCounter component
    - Create `src/features/product-catalog/ResultCounter.tsx`
    - Display "N product(s) found" format, update synchronously with product list
    - Display "0 products found" when count is zero
    - _Requirements: 7.1, 7.2, 7.3_

- [ ] 6. Implement main container and responsive layout
  - [ ] 6.1 Create ProductCatalog container component
    - Create `src/features/product-catalog/ProductCatalog.tsx`
    - Orchestrate layout: loading indicator, error display, FilterPanel, ProductList, ResultCounter
    - Apply responsive layout: multi-column (filters beside list) at ≥768px, single-column (filters above list) at <768px
    - Show loading indicator while fetching; remove within 1 second of response
    - Show error message on fetch failure
    - _Requirements: 1.2, 1.4, 9.1, 9.2_

  - [ ] 6.2 Update barrel export and wire into application
    - Update `src/features/product-catalog/index.ts` to export ProductCatalog component
    - Integrate into App.tsx or relevant route
    - _Requirements: 1.1_

- [ ] 7. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 8. Write unit and integration tests
  - [ ]\* 8.1 Write unit tests for components
    - Create `src/features/product-catalog/productCatalog.test.tsx`
    - Test ProductCard renders all fields correctly with formatted values
    - Test ProductList renders empty state messages
    - Test CategoryFilter toggles selections
    - Test PriceRangeFilter validates input and shows errors
    - Test SortControl renders all options and triggers changes
    - Test SearchInput debounces input
    - Test ResultCounter displays correct format
    - Test responsive layout switches at breakpoints
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 3.1, 4.5, 4.6, 5.1, 6.3, 7.1, 9.1, 9.2_

  - [ ]\* 8.2 Write integration tests for full filter pipeline
    - Test mounting ProductCatalog with mocked API (MSW), applying multiple filters, and verifying rendered output
    - Test fetch lifecycle: loading state → success, loading state → error, unmount cancellation
    - Test combined filter scenario: select category + set price range + enter search → verify correct products shown
    - _Requirements: 1.1, 1.2, 1.4, 1.5, 8.1, 8.2, 8.3_

- [ ] 9. Final checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties from the design document using fast-check
- Unit tests validate specific examples, edge cases, and component rendering
- All filtering, sorting, and searching happens client-side after the single API fetch
- The implementation uses TypeScript throughout with Redux Toolkit for state management

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2"] },
    { "id": 1, "tasks": ["2.1", "2.3"] },
    { "id": 2, "tasks": ["2.2"] },
    { "id": 3, "tasks": ["2.4", "2.5", "4.1", "4.2"] },
    { "id": 4, "tasks": ["5.1", "5.4", "5.5"] },
    { "id": 5, "tasks": ["5.2", "5.3"] },
    { "id": 6, "tasks": ["6.1"] },
    { "id": 7, "tasks": ["6.2"] },
    { "id": 8, "tasks": ["8.1", "8.2"] }
  ]
}
```
