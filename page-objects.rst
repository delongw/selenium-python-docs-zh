.. _page-objects:

页面对象
=========


这章是对 页面对象设计模型的特别指导。一个页面对象代表了你要测试的用户接口交互的区域。

使用页面对象模型的好处：

* 可以写出能在多个测试案例里复用的代码
* 减少重复代码
* 如果用户接口更改，只需要在一个地方做相应修改即可

测试用例
-------------

下面这个测试用例 测试了在 `python.org` 网页上搜索一个单词并确认有相应的搜索结果

::

	import unittest
	from selenium import webdriver
	import page

	class PythonOrgSearch(unittest.TestCase):
		"""A sample test class to show how page object works"""

		def setUp(self):
			self.driver = webdriver.Firefox()
			self.driver.get("http://www.python.org")

		def test_search_in_python_org(self):
			"""
			Tests python.org search feature. Searches for the word "pycon" then verified that some results show up.
			Note that it does not look for any particular text in search results page. This test verifies that
			the results were not empty.
			"""

			#Load the main page. In this case the home page of Python.org.
			main_page = page.MainPage(self.driver)
			#Checks if the word "Python" is in title
			assert main_page.is_title_matches(), "python.org title doesn't match."
			#Sets the text of search textbox to "pycon"
			main_page.search_text_element = "pycon"
			main_page.click_go_button()
			search_results_page = page.SearchResultsPage(self.driver)
			#Verifies that the results page is not empty
				assert search_results_page.is_results_found(), "No results found."

		def tearDown(self):
			self.driver.close()

	if __name__ == "__main__":
		unittest.main()


页面对象
---------------

页面对象模型旨在给每一个Web页面创造一个对象。
运用这个技术我们可以在测试代码和技术实现之间创建一个分离层，``page.py`` 会是这样的

::

	from element import BasePageElement
	from locators import MainPageLocators

	class SearchTextElement(BasePageElement):
		"""This class gets the search text from the specified locator"""

		#The locator for search box where search string is entered
		locator = 'q'


	class BasePage(object):
		"""Base class to initialize the base page that will be called from all pages"""

		def __init__(self, driver):
			self.driver = driver


	class MainPage(BasePage):
		"""Home page action methods come here. I.e. Python.org"""

		#Declares a variable that will contain the retrieved text
		search_text_element = SearchTextElement()

		def is_title_matches(self):
			"""Verifies that the hardcoded text "Python" appears in page title"""
			return "Python" in self.driver.title

		def click_go_button(self):
			"""Triggers the search"""
			element = self.driver.find_element(*MainPageLocators.GO_BUTTON)
			element.click()


	class SearchResultsPage(BasePage):
		"""Search results page action methods come here"""

		def is_results_found(self):
			# Probably should search for this text in the specific page
			# element, but as for now it works fine
			return "No results found." not in self.driver.page_source


页面元素
---------------

``element.py`` 类大致是这样的：

::

	from selenium.webdriver.support.ui import WebDriverWait

	class BasePageElement(object):
		"""Base page class that is initialized on every page object class."""

		def __set__(self, obj, value):
			"""Sets the text to the value supplied"""
			driver = obj.driver
			WebDriverWait(driver, 100).until(
				lambda driver: driver.find_element_by_name(self.locator))
			driver.find_element_by_name(self.locator).send_keys(value)

		def __get__(self, obj, owner):
			"""Gets the text of the specified object"""
			driver = obj.driver
			WebDriverWait(driver, 100).until(
				lambda driver: driver.find_element_by_name(self.locator))
			element = driver.find_element_by_name(self.locator)
			return element.get_attribute("value")


定位器
-----------

实践中, 应该把定位器的字符串根据他们的使用位置区分开.
下面例子中, 同一个页面的定位器属于同一个类.

locators.py 文件的内容是

::

	from selenium.webdriver.common.by import By

	class MainPageLocators(object):
		"""A class for main page locators. All main page locators should come here"""
		GO_BUTTON = (By.ID, 'submit')

	class SearchResultsPageLocators(object):
		"""A class for search results locators. All search results locators should come here"""
		pass
