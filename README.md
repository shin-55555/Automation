from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# ChromeDriverのパス
driver_path = r"C:/chromedriver.exe"

# ドライバー起動
service = Service(driver_path)
driver = webdriver.Chrome(service=service)

# 操作したいURLのリスト
urls = [
    "https://accounts.yahoo.co.jp/activity/access/site?done=https%3A%2F%2Fwww.yahoo.co.jp%2F",
    "https://accounts.yahoo.co.jp/activity/access/search?done=https%3A%2F%2Fwww.yahoo.co.jp%2F",
    "https://accounts.yahoo.co.jp/activity/access/ad?done=https%3A%2F%2Fwww.yahoo.co.jp%2F"
]

try:
    # URLを1つずつ処理
    for url in urls:
        print(f"アクセス中: {url}")
        driver.get(url)
        wait = WebDriverWait(driver, 30)

        while True:
            try:
                # 全選択用チェックボックスを探してクリック
                select_all_checkbox = wait.until(
                    EC.element_to_be_clickable((By.XPATH, '//*[@id="yjMain"]/div/div[1]/form/table/thead/tr/th[1]/label/span[1]'))
                )
                select_all_checkbox.click()
                time.sleep(0.5)  # UI更新待機

                # 削除ボタンが表示され、クリック可能になるまで待ってクリック(チェックボックスにチェックが入ると削除ボタンが有効化される仕様)
                delete_button = wait.until(
                    EC.element_to_be_clickable(
                        (By.XPATH, '//*[@id="yjMain"]/div/div[1]/div/button[2]/span')
                    )
                )
                if delete_button.is_enabled():
                   delete_button.click()
                   confirm_delete_button = wait.until(EC.element_to_be_clickable((By.XPATH, '//*[@id="deleteHistoryModal"]/div[2]/div[3]/a[2]')))
                   confirm_delete_button.click()
                   confirm_delete_button = wait.until(EC.element_to_be_clickable((By.XPATH, '//*[@id="completeDeleteHistoriesModal"]/div[2]/div[2]/a')))
                   confirm_delete_button.click()
                else:
                    print("削除ボタンが無効なためスキップします")


                # 削除後の反映待ち
                time.sleep(2)

            except Exception:
                # チェックボックスが見つからない、または削除ボタンがない → 終了
                print(f"{url} の処理が完了しました。次のURLへ。")
                break

except Exception as e:
    print("エラーが発生しました:", e)

finally:
    driver.quit()
    print("全てのURL処理が完了しました。ブラウザを閉じました。")
