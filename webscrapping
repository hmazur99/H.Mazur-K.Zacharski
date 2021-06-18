from pandas.core.frame import DataFrame
from selenium.webdriver import Firefox
import pandas as pd
import sqlite3

import time


def get_records() -> pd.DataFrame:
    # Uruchom przeglądarkę i przejdź do strony
    driver = Firefox()
    driver.get(
        "https://migracje.gov.pl/statystyki/zakres/polska/typ/wnioski/widok/tabele/rok/2021/")

    # Czekaj aż się załaduje
    time.sleep(10)

    # Pobierz tekst z tabeli do listy
    tablica = []
    ended = False
    while not ended:
        # Wpisz dane z tabeli na stronie do listy
        for table in driver.find_elements_by_xpath("//table[@id='DataTables_Table_0']/tbody/tr"):
            data = [int(item.text) for item in table.find_elements_by_xpath(
                ".//*[self::td or self::th]")]
            tablica.append(data)
        next_button = driver.find_element_by_id("DataTables_Table_0_next")
        # Sprawdź, czy przycisk następnej strony jest klasy "disabled" i jeśli tak - skończ iterację
        if "disabled" in next_button.get_attribute("class").split(" "):
            ended = True
        else:
            next_button.click()

    # Zamykam przeglądarkę
    driver.close()

    # Zwracamy listę przekonwertowaną na DataFrame z odpowiednimi kolumnami
    return pd.DataFrame(tablica, columns=["Wiek", "Rok", "Liczba"])


def tablica_csv(dataframe: pd.DataFrame):
    dataframe.to_csv("tablica.csv", index=False)


def tablica_sql(dataframe: pd.DataFrame):
    conn = sqlite3.connect("tablica.db")
    c = conn.cursor()
    c.execute("CREATE TABLE IF NOT EXISTS Migracje (Wiek INTEGER, Rok INTEGER, Liczba INTEGER);")
    c.execute("DELETE FROM Migracje;")
    for index, row in dataframe.iterrows():
        c.execute(f"INSERT INTO Migracje (Wiek, Rok, Liczba) VALUES ({row['Wiek']}, {row['Rok']}, {row['Liczba']});")
    conn.commit()
    conn.close()


tablica = get_records()
tablica_csv(tablica)
tablica_sql(tablica)
print(tablica)
