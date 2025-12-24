@app.route("/upload", methods=["POST"])
def upload_file():
    file = request.files["video"]

    if not file:
        return "No file uploaded"

    filename = file.filename
    file_ext = filename.rsplit(".", 1)[-1].lower()

    file_path = os.path.join(UPLOAD_FOLDER, filename)
    file.save(file_path)

    # IMAGE FILE
    if file_ext in ["jpg", "jpeg", "png", "webp"]:
        return render_template(
            "result.html",
            file_type="image",
            file_path=url_for("static", filename="uploads/" + filename)
        )

    # VIDEO FILE
    cap = cv2.VideoCapture(file_path)
    success, frame = cap.read()
    cap.release()

    thumb_name = filename.rsplit(".", 1)[0] + ".jpg"
    thumb_path = os.path.join(THUMB_FOLDER, thumb_name)

    if success:
        cv2.imwrite(thumb_path, frame)

    return render_template(
        "result.html",
        file_type="video",
        video_file=url_for("static", filename="uploads/" + filename),
        thumb_file=url_for("static", filename="thumbs/" + thumb_name)
    )
