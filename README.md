# AIC
from tkinter import *
import random
import math
from owlready2 import *
from PIL import Image, ImageTk

# ------------------- LOAD OWL ONTOLOGY -------------------
onto = get_ontology("math_surface_area.owl").load()  # Load the OWL file using owlready2

# ------------------- FORMULAS -------------------
FORMULAS = {
    "Sphere": "Surface Area = 4 × π × Radius²",  
    "Cube": "Surface Area = 6 × Side²",  
    "Cylinder": "Surface Area = 2πr(h + r)",
    "TriangularPrism": "Surface Area = 2 * base * height + perimeter * length"
}


# ------------------- MAIN WINDOW -------------------
root = Tk()
root.title("Math ITS – Surface Area Tutor")
root.geometry("800x800")
root.config(bg="#eceff1")  # Green Nature Background

# ------------------- GLOBAL VARIABLES -------------------
score = {"correct": 0, "total": 0}
current_answer = None
current_shape = None
hint_stage = 0
values = {}
formula_text, hint_text = "", ""

# ------------------- FRAME MANAGEMENT -------------------
frames = {}
shapes_list = ["Sphere", "Cube", "Cylinder", "TriangularPrism"]

def show_frame(shape):
    frames[shape].tkraise()
    ask_question(shape)

# ------------------- CREATE SHAPE FRAMES -------------------
for shape in shapes_list:
    frame = Frame(root, bg="#eceff1")
    frame.place(relwidth=1, relheight=1)
    frames[shape] = frame

    Label(frame, text=f"{shape} Surface Area Tutor", font=("Arial", 22, "bold"),
          bg="#eceff1", fg="#d32f2f").pack(pady=10)

    score_label = Label(frame, text=f"Score: {score['correct']}/{score['total']}",
                        font=("Arial", 16, "bold"), bg="#eceff1", fg="#d32f2f")
    score_label.pack(pady=5)
    frames[shape+"_score_label"] = score_label

    # Display Image for Shape
    image_path = f"images/{shape}.jpg"
    img = Image.open(image_path)
    img = img.resize((200, 200), Image.Resampling.LANCZOS)  # Corrected for Pillow >= 10
    img = ImageTk.PhotoImage(img)

    image_label = Label(frame, image=img, bg="#eceff1")
    image_label.image = img  # Keep a reference to the image
    image_label.pack(pady=10)

    q_label = Label(frame, text="", font=("Consolas", 16),
                    bg="#eceff1", fg="#d32f2f", justify=LEFT)
    q_label.pack(pady=10)
    frames[shape+"_question_label"] = q_label

    entry = Entry(frame, font=("Arial", 18), width=10, bg="white", fg="#d32f2f")
    entry.pack(pady=10)
    frames[shape+"_entry"] = entry

    feedback = Label(frame, text="", font=("Arial", 15, "bold"),
                     bg="#eceff1", fg="#d32f2f")
    feedback.pack(pady=5)
    frames[shape+"_feedback"] = feedback

    hint = Label(frame, text="", font=("Arial", 14),
                 bg="#eceff1", fg="#66bb6a", wraplength=500)
    hint.pack(pady=10)
    frames[shape+"_hint"] = hint

    btn_frame = Frame(frame, bg="#eceff1")
    btn_frame.pack(pady=10)

    Button(btn_frame, text="Check", font=("Arial", 16, "bold"),
           bg="#0d47a1", fg="white",
           command=lambda s=shape: check_answer(s)).grid(row=0, column=0, padx=10)

    next_btn = Button(btn_frame, text="Next", font=("Arial", 16),
                      bg="#0d47a1", fg="white",
                      command=lambda s=shape: ask_question(s))
    next_btn.grid(row=0, column=1, padx=10)
    next_btn.grid_remove()
    frames[shape+"_next_btn"] = next_btn

    Button(frame, text="Hint", font=("Arial", 16),
           bg="#0d47a1", fg="white",
           command=lambda s=shape: give_hint(s)).pack(pady=5)

    Button(frame, text="Back to Menu", font=("Arial", 14),
           bg="#0d47a1", fg="white",
           command=lambda: show_main_menu()).pack(pady=5)

# ------------------- MAIN MENU -------------------
main_menu = Frame(root, bg="#e8f5e9")
main_menu.place(relwidth=1, relheight=1)

Label(main_menu, text="Math ITS – Surface Area Tutor",
      font=("Arial", 23, "bold"),
      bg="#e8f5e9", fg="#d32f2f").pack(pady=20)

menu_text = (
    "Welcome! Choose a 3D shape and practice calculating the surface area.\n"
    "Includes hints, steps, and scoring.\n"
)
Label(main_menu, text=menu_text, font=("Arial", 14),
      bg="#e8f5e9", fg="#d32f2f").pack(pady=10)

tip_label = Label(main_menu, font=("Arial", 14, "italic"),
                  bg="#e8f5e9", fg="#66bb6a")
tip_label.pack(pady=10)

for s in shapes_list:
    Button(main_menu, text=s, font=("Arial", 14),
           bg="#0d47a1", fg="white", width=20,
           command=lambda s=s: show_frame(s)).pack(pady=5)

def show_main_menu():
    main_menu.tkraise()


# ------------------- QUESTION GENERATOR -------------------
def generate_question(shape_name):
    global formula_text, hint_text
    shape = onto.search_one(iri=f"*{shape_name}")
    dims = {}

    if shape_name == "Sphere":
        R = random.randint(2,12)
        dims = {"R": R}
        answer = 4 * math.pi * R * R
    elif shape_name == "Cube":
        S = random.randint(2, 12)
        dims = {"S": S}
        answer = 6 * S * S
    elif shape_name == "Cylinder":
        R = random.randint(2, 12)
        H = random.randint(2, 12)
        dims = {"R": R, "H": H}
        answer = 2 * math.pi * R * (R + H)
    elif shape_name == "TriangularPrism":
        B = random.randint(2, 12)
        H = random.randint(2, 12)
        L = random.randint(2, 12)
        dims = {"B": B, "H": H, "L": L}
        answer = 2 * B * H + (B + B + H) * L

    # Fetch the formula and hint from the ontology
    formula_text = FORMULAS.get(shape_name, "No formula available")
    hint_text = str(shape.hasHint[0]) if shape.hasHint else "No hint available"

    return dims, answer

def ask_question(shape):
    global current_answer, current_shape, hint_stage, values
    current_shape = shape
    hint_stage = 0
    frames[shape+"_feedback"].config(text="")
    frames[shape+"_hint"].config(text="")
    frames[shape+"_entry"].delete(0, END)
    frames[shape+"_next_btn"].grid_remove()

    values, current_answer = generate_question(shape)
    # Question formatting with dimensions shown as in the desired format
    dims_str = "\n".join([f"{k}: {v}" for k, v in values.items()])
    frames[shape+"_question_label"].config(
        text=f"{shape}\n{'-'*len(shape)}\n{dims_str}"
    )

# ------------------- HINT SYSTEM -------------------
def give_hint(shape):
    global hint_stage
    if current_shape != shape:
        return
    if hint_stage == 0:
        frames[shape+"_hint"].config(text=f"📘 Formula:\n{formula_text}")
    elif hint_stage == 1:
        frames[shape+"_hint"].config(text=f"📘 Steps:\nSubstitute values: {values}")
    elif hint_stage == 2:
        frames[shape+"_hint"].config(text=f"📘 Answer:\n= {current_answer}")
    else:
        frames[shape+"_hint"].config(text="No more hints available.")
    hint_stage += 1

# ------------------- CHECK ANSWER -------------------
CORRECT_FEEDBACK = ["Excellent! ✅", "Great job! 🌟", "Perfect! 👍"]

def check_answer(shape):
    global score
    try:
        user_value = float(frames[shape+"_entry"].get())
        score["total"] += 1
        if abs(user_value - current_answer) < 0.01:  # tolerance for decimals
            score["correct"] += 1
            frames[shape+"_feedback"].config(
                text=random.choice(CORRECT_FEEDBACK), fg="#d32f2f"
            )
        else:
            frames[shape+"_feedback"].config(
                text=f"❌ Incorrect! Correct answer = {current_answer}", fg="#c62828"
            )
        frames[shape+"_score_label"].config(
            text=f"Score: {score['correct']}/{score['total']}"
        )
        frames[shape+"_next_btn"].grid()
    except:
        frames[shape+"_feedback"].config(
            text="Enter a valid number!", fg="#c62828"
        )

# ------------------- START APP -------------------
show_main_menu()
root.mainloop()
