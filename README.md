# jason
import requests
import re
from datetime import datetime

# 下载文本文档
def download_text(url):
    response = requests.get(url)
    if response.status_code == 200:
        return response.text
    else:
        return None

# 检查是否存在特定的关键词
def check_keywords(text, keywords, log_file):
    matches_server = re.findall(r'object-group network server([\s\S]*?)object-group network', text)
    for match in matches_server:
        for keyword in keywords:
            # 检查是否包含特定词汇
            if re.search(rf'\b{keyword}\b', match):
                ip_references = re.findall(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', match)
                log_file.write(f"Warning: '{keyword}' found in 'object-group network server' section.\n")
                log_file.write("Related IP references: " + ', '.join(ip_references) + "\n")

    matches_temp = re.findall(r'object-group network temp([\s\S]*?)object-group network', text)
    for match in matches_temp:
        ips = re.findall(r'\b(?:\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})\b', match)
        for ip in ips:
            if ip != '1.1.1.1':
                log_file.write("Warning: IP found in 'object-group network temp' section, excluding 1.1.1.1.\n")
                log_file.write("Related IP reference: " + ip + "\n")

# 主函数
def main():
    url_keywords = {
        'URL1': ['word1', 'word2', 'word3'],
        'URL2': ['word1', 'word2'],
        # 添加更多的 URL 和关键词
    }

    log_file_path = r"D:\check_word_log.txt"
    with open(log_file_path, 'a') as log_file:
        log_file.write(f"Log generated on {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n")

        for url, keywords in url_keywords.items():
            # 下载文本文档
            document_text = download_text(url)

            if document_text:
                # 检查关键词
                check_keywords(document_text, keywords, log_file)
            else:
                log_file.write(f"Failed to download the document from {url}.\n")

if __name__ == "__main__":
    main()
