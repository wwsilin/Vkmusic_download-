# Vkmusic_download-
Скрипт для качки музла из ВК на базе либы VKpyMusic

## Зависимости 
Нужно установить либу https://github.com/issamansur/vkpymusic/tree/main\
```bash
 $ pip install vkpymusic
```
## Использование
Копируем и сохраняем код
```python
# -*- coding: utf-8 -*-
from vkpymusic import Service, TokenReceiver
import sys
import locale
import os

# Настройка кодировки для разных платформ
try:
    sys.stdin.reconfigure(encoding='utf-8')
    sys.stdout.reconfigure(encoding='utf-8')
except AttributeError:
    # Для старых версий Python
    pass

locale.setlocale(locale.LC_ALL, 'ru_RU.UTF-8')

# Для Windows CMD
if os.name == 'nt':
    os.system('chcp 65001 > nul')

def safe_input(prompt):
    """Безопасный ввод с обработкой кодировки"""
    print(prompt, end='', flush=True)
    try:
        return sys.stdin.buffer.readline().decode('utf-8').strip()
    except UnicodeDecodeError:
        return sys.stdin.readline().strip()

def auth_user():
    """Аутентификация пользователя"""
    print("\n--- Аутентификация ---")
    login = safe_input("Введите логин VK: ")
    password = safe_input("Введите пароль VK: ")
    
    token_receiver = TokenReceiver(login, password)
    
    if not token_receiver.auth():
        print("\n❌ Ошибка аутентификации! Проверьте логин/пароль.")
        exit()
    
    token = token_receiver.get_token()
    token_receiver.save_to_config()
    print("\n✅ Успешная аутентификация!")
    return Service.parse_config()

def show_tracks(tracks):
    """Отображение списка треков"""
    print("\n🔍 Результаты поиска:")
    for i, track in enumerate(tracks, 1):
        print(f"{i:2}. {track.artist} - {track.title} ({track.duration})")

def download_track(service, track):
    """Загрузка трека"""
    try:
        filename = service.save_music(track)
        print(f"\n✅ Успешно загружено: {track.artist} - {track.title}")
        print(f"📁 Файл: {filename}")
    except Exception as e:
        print(f"\n❌ Ошибка загрузки: {str(e)}")

def search_loop(service):
    """Основной цикл поиска"""
    while True:
        try:
            query = safe_input("\n🎵 Введите название трека (или 'q' для выхода): ")
            
            if query.lower() == 'q':
                print("\n👋 До свидания!")
                exit()
                
            if not query.strip():
                print("\n⚠️ Введите непустой запрос!")
                continue
                
            tracks = service.search_songs_by_text(query, count=10)
            
            if not tracks:
                print("\n😞 Ничего не найдено. Попробуйте другой запрос.")
                continue
                
            show_tracks(tracks)
            
            while True:
                choice = safe_input("\n🔢 Выберите номер трека (1-10) или 'b' для возврата: ").strip().lower()
                
                if choice == 'b':
                    break
                if choice.isdigit():
                    index = int(choice) - 1
                    if 0 <= index < len(tracks):
                        download_track(service, tracks[index])
                        break
                    print("\n⚠️ Некорректный номер. Введите число от 1 до", len(tracks))
                else:
                    print("\n⚠️ Некорректный ввод. Используйте цифры или 'b'")
                    
        except KeyboardInterrupt:
            print("\n\n👋 До свидания!")
            exit()

def main():
    """Главная функция"""
    try:
        service = Service.parse_config()
        print("\n🔑 Используется сохранённый токен")
    except Exception as e:
        print(f"\n⚠️ Ошибка загрузки конфига: {str(e)}")
        service = auth_user()
        
    print("\n" + "="*40)
    print("🎧 VK Music Downloader")
    print("="*40)
    print("🔍 Поиск треков по названию")
    print("⏎ Для выхода в любой момент нажмите Ctrl+C или введите 'q'")
    
    try:
        search_loop(service)
    except Exception as e:
        print(f"\n❌ Критическая ошибка: {str(e)}")
        exit()

if __name__ == "__main__":
    main()
```
Запускаем и радуемся!
