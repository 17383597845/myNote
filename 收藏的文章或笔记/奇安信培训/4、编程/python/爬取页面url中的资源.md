```
import os
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin, urlparse

def download_url(url, folder):
    response = requests.get(url)
    if response.status_code == 200:
        # 获取页面内容
        html_content = response.text
        soup = BeautifulSoup(html_content, 'html.parser')

        # 获取页面中的所有链接
        links = [a['href'] for a in soup.find_all('a', href=True)]

        # 创建保存资源的文件夹
        if not os.path.exists(folder):
            os.makedirs(folder)

        # 下载链接中的资源
        for link in links:
            absolute_url = urljoin(url, link)
            filename = os.path.join(folder, os.path.basename(absolute_url))

            try:
                response = requests.get(absolute_url)
                with open(filename, 'wb') as file:
                    file.write(response.content)
                print(f"下载成功: {filename}")
            except Exception as e:
                print(f"下载失败: {absolute_url}, 错误: {e}")

if __name__ == "__main__":
    # 要爬取的页面URL
    target_url = "https://example.com"

    # 保存资源的文件夹
    download_folder = "downloaded_resources"

    # 调用函数进行爬取和下载
    download_url(target_url, download_folder)

```


```
import os
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin, urlparse

def download_resources(url, folder):
    # 发送GET请求获取页面内容
    response = requests.get(url)
    if response.status_code == 200:
        # 使用BeautifulSoup解析HTML内容
        soup = BeautifulSoup(response.text, 'html.parser')

        # 提取所有链接的URL
        urls = [urljoin(url, link.get('href')) for link in soup.find_all('a', href=True)]

        # 创建保存资源的文件夹
        if not os.path.exists(folder):
            os.makedirs(folder)

        # 下载链接中的资源
        for resource_url in urls:
            # 使用urlparse获取资源的文件名
            resource_filename = os.path.basename(urlparse(resource_url).path)

            if resource_filename:  # 排除目录链接
                try:
                    response = requests.get(resource_url)
                    with open(os.path.join(folder, resource_filename), 'wb') as file:
                        file.write(response.content)
                    print(f"下载成功: {resource_filename}")
                except Exception as e:
                    print(f"下载失败: {resource_url}, 错误: {e}")
            else:  # 处理子目录链接
                subfolder_name = os.path.basename(urlparse(resource_url).path)
                subfolder_path = os.path.join(folder, subfolder_name)
                download_resources(resource_url, subfolder_path)

if __name__ == "__main__":
    # 要爬取的页面URL
    target_url = "https://example.com"

    # 保存资源的文件夹
    download_folder = "downloaded_resources"

    # 调用函数进行爬取和下载
    download_resources(target_url, download_folder)

```