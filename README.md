# pypropertysearch
Python property search tool using live data from Zillow API.
# Hermela Abera Property Search tool
import requests

# Function to get input from the user
def get_user_input():
    print("Welcome to the Property Search Tool!")
    zip_code = input("Enter the zip code/ city: ").strip()
    status_type = input("Do you want to rent or buy? (Enter 'ForRent' or 'ForSale'): ").strip()
    property_type = input("What type of property? (Houses, Apartments, Condos): ").strip()
    bedrooms = int(input("Enter the exact number of bedrooms you want: ").strip())
    return zip_code, status_type, property_type, bedrooms

# Function to fetch data from the API
def fetch_properties(zip_code, status_type, property_type):
    print("\nLoading property data...")
    url = "https://zillow-com1.p.rapidapi.com/propertyExtendedSearch"
    headers = {
        "x-rapidapi-host": "zillow-com1.p.rapidapi.com",
        "x-rapidapi-key": "ENTER YOUR API KEY HERE! "
    }
    params = {
        "location": zip_code,
        "status_type": status_type,
        "home_type": property_type,
        "resultsPerPage": 200
    }
    response = requests.get(url, headers=headers, params=params) 
    
    if response.status_code == 200: # checks the status of the API data coming in
        data = response.json()
        print(data)
        return data.get("props", []) # gets values associated with props(key) of many houses 
    else:
        print("Failed to fetch data. Please check your inputs or try again later.")
        return []

# Function to filter properties by number of bedrooms
def filter_properties_by_bedrooms(properties, bedrooms):
    filtered = []
    for property in properties:
        if property.get("bedrooms") == bedrooms:
            filtered.append(property)
    return filtered

# Function to calculate the median price
def calculate_median_price(properties):
  prices = []
    for property in properties:
        price = property.get("price")
        # Add price to the list only if it is a valid number
        if price and type(price) in [int, float]:
            prices.append(price)
    if len(prices) == 0:
        return 0
    
    prices.sort() # sort to calculate median
    
    middle = len(prices) // 2
    
    if len(prices) % 2 == 1: # if len(prices) is odd print middle
        return prices[middle]
    else:
        return (prices[middle - 1] + prices[middle]) / 2 # if len(prices) is even add the middle and middle minus 1 divide by 2 

# Function to display the properties
def display_properties(properties):
    print("\nProperties matching your search:")
    print(f"{'Address':<50} {'Price':<10} {'Beds':<5}") # space in between 
    print("-" * 65) # dotted line to separate titles
    for property in properties:
        address = property.get("address", "N/A")
        price = property.get("price")
        price = f"${price:,.2f}" # format number with dollar sign comma and decimal
        bedrooms = property.get("bedrooms", "N/A")

        print(f"{address:<50} {price:<10} {bedrooms:<5}")
# Main function to run the program
def main():
    zip_code, status_type, property_type, bedrooms = get_user_input() # gets user preferences
    
    all_properties = fetch_properties(zip_code, status_type, property_type)   # takes zipcode/City , Rent/Sale , Apartment or House from the API
    
    filtered_properties = filter_properties_by_bedrooms(all_properties, bedrooms) # filters the specified locations properties by num bedrooms
    
    if len(filtered_properties) > 0:
        
        display_properties(filtered_properties)
        median_price = calculate_median_price(filtered_properties)
        
        if status_type == "ForRent":
            print(f"\nMedian Rent Price: ${median_price:,.2f}")
        else:
            print(f"\nMedian Sale Price: ${median_price:,.2f}")
    else:
        print("\nNo properties found matching your criteria.")

# Run the program
if __name__ == "__main__":
    main()
        
