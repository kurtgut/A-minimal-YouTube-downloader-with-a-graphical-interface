# A-minimal-YouTube-downloader-with-a-graphical-interface
# Версия 0.1.0
# программа дя скачивания видео с Ютуба в лучшем качестве
# Скачивает 1 видео
# скеивает видео со звуком
import yt_dlp
import tkinter as tk
from tkinter import messagebox, filedialog
import shutil

def download_video():
    url = entry.get().strip()
    if not url:
        messagebox.showwarning("Предупреждение", "Введите ссылку на видео!")
        return

    # проверка ffmpeg
    if shutil.which("ffmpeg") is None:
        messagebox.showerror("Ошибка", "Не найден ffmpeg! Установите его и добавьте в PATH.")
        return

    # выбор папки для сохранения
    folder = filedialog.askdirectory(title="Выберите папку для сохранения")
    if not folder:
        return

    # параметры yt-dlp
    ydl_opts = {
        "format": "bestvideo+bestaudio/best",
        "outtmpl": f"{folder}/%(title)s.%(ext)s",
        "extractor_args": {"youtube": ["player_client=default"]},  # обход без JS-runtime
        "merge_output_format": "mp4",
        "noplaylist": True  # <--- важно! только одно видео
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])
        messagebox.showinfo("Успех", "Видео успешно загружено!")
    except Exception as e:
        messagebox.showerror("Ошибка загрузки", f"Не удалось загрузить видео:\n{e}")

def show_context_menu(event):
    context_menu.tk_popup(event.x_root, event.y_root)

# GUI
root = tk.Tk()
root.title("YouTube Загрузчик (yt-dlp)")

tk.Label(root, text="Введите URL видео:").pack(pady=5)
entry = tk.Entry(root, width=60)
entry.pack(pady=5)

context_menu = tk.Menu(root, tearoff=0)
context_menu.add_command(label="Копировать", command=lambda: root.clipboard_append(entry.selection_get()))
context_menu.add_command(label="Вставить", command=lambda: entry.insert(tk.INSERT, root.clipboard_get()))
entry.bind("<Button-3>", show_context_menu)

tk.Button(root, text="Загрузить видео", command=download_video).pack(pady=10)

root.mainloop()
