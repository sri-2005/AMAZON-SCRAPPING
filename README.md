    # AMAZON-SCRAPPING using python 
    # here i created this using beautifull soup, pandas and numpy
    
    
    from bs4 import BeautifulSoup
    import requests
    import pandas as pd
    import numpy as np
    def get_title(soup):
        try:
            title = soup.find("span", attrs={"id": 'productTitle'})
            title_string = title.text.strip() if title else ""
        except AttributeError:
            title_string = ""
        return title_string
    
    
    def get_price(soup):
        try:
            price = soup.find("span", attrs={'id': 'priceblock_ourprice'}) or \
                    soup.find("span", attrs={'id': 'priceblock_dealprice'})
            return price.text.strip() if price else "N/A"
        except AttributeError:
            return "N/A"
    
    
    def get_rating(soup):
        try:
            rating = soup.find("span", attrs={'class': 'a-icon-alt'})
            return rating.text.strip() if rating else "N/A"
        except AttributeError:
            return "N/A"
    
    
    def get_review_count(soup):
        try:
            review_count = soup.find("span", attrs={'id': 'acrCustomerReviewText'})
            return review_count.text.strip() if review_count else "N/A"
        except AttributeError:
            return "N/A"
    
    
    def get_availability(soup):
        try:
            available = soup.find("div", attrs={'id': 'availability'})
            return available.find("span").text.strip() if available else "Not Available"
        except AttributeError:
            return "Not Available"
    if __name__ == '__main__':
        HEADERS = {
            'User-Agent': 'your-user-agent-here',  # Update this to a valid User-Agent string
            'Accept-Language': 'en-US, en;q=0.5'
        }
    
    # The webpage URL
    URL = "https://www.amazon.com/s?k=boat+airpods"
    
   
    webpage = requests.get(URL, headers=HEADERS)
    soup = BeautifulSoup(webpage.content, "html.parser")

   
    links = soup.find_all("a", attrs={'class': "a-link-normal s-no-outline"})  

    # Store the links
    links_list = [link.get('href') for link in links]

    # Data dictionary to store product details
    d = {"title": [], "price": [], "rating": [], "reviews": [], "availability": []}

    # Loop for extracting product details from each link 
    for link in links_list:
        new_webpage = requests.get("https://www.amazon.com" + link, headers=HEADERS)
        new_soup = BeautifulSoup(new_webpage.content, "html.parser")

        # Function calls to display all necessary product information
        d['title'].append(get_title(new_soup))
        d['price'].append(get_price(new_soup))
        d['rating'].append(get_rating(new_soup))
        d['reviews'].append(get_review_count(new_soup))
        d['availability'].append(get_availability(new_soup))

    # Creating a DataFrame and saving it as a CSV
    amazon_df = pd.DataFrame.from_dict(d)
    amazon_df['title'].replace('', np.nan, inplace=True)
    amazon_df = amazon_df.dropna(subset=['title'])
    amazon_df.to_csv("amazon_boat.csv", header=True, index=False)
    print(amazon_df)

