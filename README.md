# Tiki-Web-Scraping-with-Selenium

Build a web-crawler that take in a Tiki URL and return a dataframe contains information of products.

##### Install resources
##### install selenium and other resources for crawling data

!pip install selenium
!apt-get update
!apt install chromium-chromedriver

##### Import necessary libraries

import re
import time
import pandas as pd

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException

##### Configuration for Driver and links

##### Urls
TIKI = 'https://tiki.vn'

MAIN_CATEGORIES = [
    {'Name': 'Điện Thoại - Máy Tính Bảng',
     'URL': 'https://tiki.vn/dien-thoai-may-tinh-bang/c1789'},

    {'Name': 'Điện Tử - Điện Lạnh',
     'URL': 'https://tiki.vn/tivi-thiet-bi-nghe-nhin/c4221'},

    {'Name': 'Phụ Kiện - Thiết Bị Số', 
     'URL': 'https://tiki.vn/thiet-bi-kts-phu-kien-so/c1815'},

    {'Name': 'Laptop - Thiết bị IT', 
     'URL': 'https://tiki.vn/laptop-may-vi-tinh/c1846'},

    {'Name': 'Máy Ảnh - Quay Phim', 
     'URL': 'https://tiki.vn/may-anh/c1801'},

    {'Name': 'Điện Gia Dụng', 
     'URL': 'https://tiki.vn/dien-gia-dung/c1882'},

    {'Name': 'Nhà Cửa Đời Sống', 
     'URL': 'https://tiki.vn/nha-cua-doi-song/c1883'},

    {'Name': 'Hàng Tiêu Dùng - Thực Phẩm', 
     'URL': 'https://tiki.vn/bach-hoa-online/c4384'},

    {'Name': 'Đồ chơi, Mẹ & Bé', 
     'URL': 'https://tiki.vn/me-va-be/c2549'},

    {'Name': 'Làm Đẹp - Sức Khỏe', 
     'URL': 'https://tiki.vn/lam-dep-suc-khoe/c1520'},

    {'Name': 'Thể Thao - Dã Ngoại', 
     'URL': 'https://tiki.vn/the-thao/c1975'},

    {'Name': 'Xe Máy, Ô tô, Xe Đạp', 
     'URL': 'https://tiki.vn/o-to-xe-may-xe-dap/c8594'},

    {'Name': 'Hàng quốc tế', 
     'URL': 'https://tiki.vn/hang-quoc-te/c17166'},

    {'Name': 'Sách, VPP & Quà Tặng', 
     'URL': 'https://tiki.vn/nha-sach-tiki/c8322'},

    {'Name': 'Voucher - Dịch Vụ - Thẻ Cào', 
     'URL': 'https://tiki.vn/voucher-dich-vu/c11312'}
]

##### Function to Start and Close Driver

##### Global driver to use throughout the script
DRIVER = None

##### Wrapper to close driver if its created
def close_driver():
    global DRIVER
    if DRIVER is not None:
        DRIVER.close()
    DRIVER = None

##### Function to (re)start driver
def start_driver(force_restart=False):
    global DRIVER
    
    if force_restart:
        close_driver()
    
##### Setting up the driver
    options = webdriver.ChromeOptions()
##### We don't want a chrome browser opens, so it will run in the background    
    options.add_argument('-headless') 
    options.add_argument('-no-sandbox')
    options.add_argument('-disable-dev-shm-usage')

    DRIVER = webdriver.Chrome('chromedriver',options=options)
    
Function to get info from one product

### NOTE: Sometimes, the web element returned by the driver can be faulty due to the way the website is set up. This can lead to a situation where calling `.text` from that web element returns an empty string, even though there are visible texts inside the element when checked manually using Inspect. When this happens, you can use `.get_attribute('innerHTML')` instead of `.text`.    

start_driver()
DRIVER.get(MAIN_CATEGORIES[6]['URL'])
DRIVER.current_url

all_products = DRIVER.find_elements(By.CLASS_NAME, 'product-item')
len(all_products)

#####  Product in the 6th position in page 1
product_item = all_products[6]
name_class = product_item.find_element(By.CLASS_NAME, 'name')
info = name_class.find_element(By.TAG_NAME, 'span').get_attribute('innerHTML')
print(info)

##### Function to extract product info from the product
def get_product_info_single(product_item):
    info = {'name':'',
            'price':'',
            'product_url':'',
            'image':''}
            
##### Get name through find_element_by_class_name
    try:
        name_class = product_item.find_element(By.CLASS_NAME, 'name')
        info['name'] = name_class.find_element(By.TAG_NAME, 'span').get_attribute('innerHTML')
    except NoSuchElementException:
        pass

##### Get price find_element_by_class_name
    try:
        info['price'] = product_item.find_element(By.CLASS_NAME, 'price-discount__price').get_attribute('innerHTML')
    except NoSuchElementException:
        info['price'] = -1
    
##### Get link from .get_attribute()
    try:
###### product_link        = : not use 
        info['product_url'] = product_item.get_attribute('href')
###### Thêm https nếu link ko hoàn chỉnh 
###### The find() method returns -1 if the value is not found.
        if info['product_url'].find("https")== -1:    
            while info['product_url'][0].isalnum() == False: 
                  info['product_url'] = info['product_url'][1:]
                  info['product_url'] = "https://" + info['product_url']
    except NoSuchElementException:
        pass
##### Get thumbnail by class_name and Tag_name and get_attribute()
    try:
        thumbnail     = product_item.find_element(By.TAG_NAME, 'picture')
        info['image'] = thumbnail.find_element(By.TAG_NAME, 'img').get_attribute('src')
    except NoSuchElementException:
        pass

##### Get badge Freeship + TikiFast

###### Using "find_element_by_css_selector() driver method – Selenium Python"
###### With this strategy, the first element with the matching CSS_selector will be returned. 
###### If no element has a matching CSS_selector, a NoSuchElementException will be raised.

######  get element
    try: 
        freeship = product_item.find_element_by_css_selector('div.thumbnail > img').get_attribute('src') 
        if freeship[-9:] == '03338.png': 
            info['freeship'] = freeship
        else: 
            info['freeship'] = 'no' 
###### Bage Freeship + Tikifast nằm ở tag <img> đầu tiên dưới class thumbnail but not always the case
###### Sau khi lấy link ảnh cần làm thêm thao tác check xem có đúng đấy là ảnh TikiFast ko 
    except NoSuchElementException:
        info['freeship'] = 'no'

###### Get percentages of star: 
    try: 
        star_location = product_item.find_element(By.CLASS_NAME,'average').get_attribute('style')
###### Number + %  , e.g. 86%        
        info['star_percent']= star_location[-5:-1]    
###### Chỉ lấy phần trăm rating, ko lấy toàn bộ text 
    except NoSuchElementException:
        info['star_percent'] = 'no info'
 
###### Get badge UnderPrice: 
    try: 
        info['underprice'] = product_item.find_element_by_css_selector('div.badge-under-price > div.item > img').get_attribute('src')
    except NoSuchElementException:
        info['underprice'] = 'no'

###### Get price discount: 
    try: 
        info['discount'] = product_item.find_element(By.CLASS_NAME, 'price-discount__discount').get_attribute('innerHTML')
    except NoSuchElementException:
        info['discount'] = 'no'
    
###### Get installment: 
    try: 
        info['installment']= product_item.find_element_by_css_selector('div.badge-benefits > div.item > span').get_attribute('innerHTML')
    except NoSuchElementException:
        info['installment'] = 'no'
    
###### Get free gifts: 
    try: 
         info['free_gifts']= product_item.find_element_by_css_selector('div.freegift-list > span').get_attribute('innerHTML')
    except NoSuchElementException:
        info['free_gifts'] = 'no'

    return info 
    
##### Function to scrape info of all products from a Page URL    
##### To make your own life easier, you should use the function for a single product inside this one.

from selenium.common.exceptions import StaleElementReferenceException
##### Function to scrape all products from a page
def get_product_info_from_page(page_url):
    """ Extract info from all products of a specfic page_url on Tiki website
        Args:
            page_url: (string) url of the page to scrape
        Returns:
            data: list of dictionary of products info. If no products shown, return empty list.
    """
    global DRIVER
###### Store the info dictionary of each product in this list
    data = []  
###### Use the driver to get info from the product page    
    DRIVER.get(page_url)
    DRIVER.current_url
###### MUST have the sleep function    
    time.sleep(2)       
    all_products = DRIVER.find_elements(By.CLASS_NAME, 'product-item')
    print(f'Found {len(all_products)} products')

    for product in all_products:
###### Look through the product and get the data
     
        try:
            info_item = get_product_info_single(product)
        except StaleElementReferenceException: 
            pass
        data.append(info_item)
        
    return data

##### Start scraping

##### FUNCTION TO CHECK IF THE NUMBER OF PAGES REQUIRED IS VALID OR NOT: 

##### Python ko báo lỗi nếu số trang vượt quá số trang sản phẩm thực tế
##### Tuy nhiên khi đó function get_product_info_from_page()sẽ không tìm được sản phẩm nào 

def check_no_of_page(no_of_page): 
######  # Example about this: https://tiki.vn/nha-cua-doi-song/c1883?page=2
    last_page_url = main_cat['URL']+'?page='+str(no_of_page)     
    data = get_product_info_from_page(last_page_url)
    return len(data)
    
main_cat  = MAIN_CATEGORIES[6] # Pick any category you like by changing the index
print(main_cat)
start_driver(force_restart=True)
print('Scraping', main_cat['Name'])
print('Main link:', main_cat['URL'])

prod_data = [] #Lưu list cộng gộp của tất cả các trang 
###### prod_data dùng trong trường hợp muốn làm nhiều trang web 
###### prod_data[data_1, data_2,...]

no_of_page = 3 # number of pages to scrap 
result_check = check_no_of_page(no_of_page)

data_list = [x for x in range(no_of_page + 1)] #Lưu list theo từng trang 

if result_check != 0: 
    print("The number of pages required is valid!")
    for num in range(1,no_of_page + 1):
###### Create page link for each page: 
        page_url = main_cat['URL']+'?page='+str(num)
        print(page_url)
###### Get data per page and save in the data_list 
        data_per_page = get_product_info_from_page(page_url)
        data_list[num] = data_per_page.copy()
        
###### Add data per page into the total list: 
        prod_data.extend(data_per_page)
else: 
    print("The number of pages required exceeds the number of available pages!")

print(f'Found {len(prod_data)} products in total')


close_driver() # Close driver when we're done

###### Run cell below to save your scraped data into a .csv file
###### If you've scraped correctly, then the cell should run without error and the information in the table should look reasonable.

##### SAVE DATA TO CSV FILE - ALL DATA 
df = pd.DataFrame(data=prod_data, columns=prod_data[0].keys())
df.to_csv('tiki_products.csv')

n_products_to_view = 10 # Change this as you like to check more products
df.head(n_products_to_view)

##### SAVE DATA TO CSV FILE - EACH PAGE: 
###### Print list by page 1,2,3...
df = pd.DataFrame(data=data_list[2], columns=data_list[2][0].keys()) 
df.to_csv('tiki_products_per_page.csv')

n_products_to_view = 10 # Change this as you like to check more products
df.head(n_products_to_view)
    

