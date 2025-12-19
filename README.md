# A-minimal-YouTube-downloader-with-a-graphical-interface
# Версия 0.1.0
# программа дя скачивания видео с Ютуба в лучшем качестве
# Скачивает 1 видео
# скеивает видео со звуком
import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import shutil
import yt_dlp

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

    # колбэк для обновления прогресса
    def progress_hook(d):
        if d['status'] == 'downloading':
            percent = d.get('_percent_str', '0.0%')
            # убираем знак % и переводим в число
            try:
                value = float(percent.replace('%', '').strip())
            except:
                value = 0
            progress_var.set(value)
            progress_label.config(text=f"Загрузка: {percent}")
            root.update_idletasks()
        elif d['status'] == 'finished':
            progress_var.set(100)
            progress_label.config(text="Загрузка завершена!")

    # параметры yt-dlp
    if mode.get() == "video":
        quality = video_quality.get()
        if quality == "best":
            fmt = "bestvideo+bestaudio/best"
        else:
            fmt = f"bestvideo[height={quality}]+bestaudio/best"

        ydl_opts = {
            "format": fmt,
            "outtmpl": f"{folder}/%(title)s.%(ext)s",
            "extractor_args": {"youtube": ["player_client=default"]},
            "merge_output_format": "mp4",
            "noplaylist": True,
            "progress_hooks": [progress_hook]
        }
    else:  # аудио
        bitrate = audio_quality.get()
        ydl_opts = {
            "format": "bestaudio/best",
            "outtmpl": f"{folder}/%(title)s.%(ext)s",
            "extractor_args": {"youtube": ["player_client=default"]},
            "noplaylist": True,
            "postprocessors": [{
                "key": "FFmpegExtractAudio",
                "preferredcodec": "mp3",
                "preferredquality": bitrate
            }],
            "progress_hooks": [progress_hook]
        }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])
        messagebox.showinfo("Успех", f"{'Видео' if mode.get()=='video' else 'Аудио'} успешно загружено!")
    except Exception as e:
        messagebox.showerror("Ошибка загрузки", f"Не удалось загрузить:\n{e}")

def show_context_menu(event):
    context_menu.tk_popup(event.x_root, event.y_root)

# GUI
root = tk.Tk()
root.title("YouTube Загрузчик (yt-dlp)")

tk.Label(root, text="Введите URL видео:").pack(pady=5)
entry = tk.Entry(root, width=60)
entry.pack(pady=5)

# выбор режима
mode = tk.StringVar(value="video")
tk.Radiobutton(root, text="Видео (MP4)", variable=mode, value="video").pack(anchor="w")
tk.Radiobutton(root, text="Аудио (MP3)", variable=mode, value="audio").pack(anchor="w")

# выбор качества видео
tk.Label(root, text="Качество видео:").pack(anchor="w")
video_quality = ttk.Combobox(root, values=["360", "720", "1080", "best"], state="readonly")
video_quality.set("best")
video_quality.pack(anchor="w")

# выбор качества аудио
tk.Label(root, text="Качество аудио (битрейт):").pack(anchor="w")
audio_quality = ttk.Combobox(root, values=["128", "192", "320"], state="readonly")
audio_quality.set("192")
audio_quality.pack(anchor="w")

# прогрессбар
progress_var = tk.DoubleVar()
progress_bar = ttk.Progressbar(root, variable=progress_var, maximum=100)
progress_bar.pack(fill="x", padx=10, pady=5)

progress_label = tk.Label(root, text="Ожидание загрузки...")
progress_label.pack()

# контекстное меню
context_menu = tk.Menu(root, tearoff=0)
context_menu.add_command(label="Копировать", command=lambda: root.clipboard_append(entry.selection_get()))
context_menu.add_command(label="Вставить", command=lambda: entry.insert(tk.INSERT, root.clipboard_get()))
entry.bind("<Button-3>", show_context_menu)

tk.Button(root, text="Загрузить", command=download_video).pack(pady=10)

root.mainloop()
