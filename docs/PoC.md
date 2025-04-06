# *Proof of Concept (PoC) - سیستم پایش آسیب سازند با هوش مصنوعی و شبیه‌سازی ۳D*

این سند یک برنامه‌ی ۴ هفته‌ای برای پیاده‌سازی یک *نمونه اولیه (PoC)* از سیستم پایش آسیب‌سازند را ارائه می‌دهد. هدف اصلی اثبات امکان‌سنجی فناوری‌های پیشنهادی (هوش مصنوعی، شبیه‌سازی و نمایش داده) است.

## *۱. اهداف کلیدی PoC*
- ✅ اثبات دقت *مدل‌های ML* در پیش‌بینی ۴ عامل کلیدی آسیب‌سازند  
- ✅ نمایش *شبیه‌سازی ۳D* از رفتار سازند تحت تنش  
- ✅ راه‌اندازی *داشبورد مدیریتی* با هشدارهای بلادرنگ  
- ✅ ارزیابی *کارایی سخت‌افزاری* برای پردازش داده‌های حجیم  

---

## *۲. طرح اجرایی (۴ هفته‌ای)*
### *📅 هفته ۱: آماده‌سازی داده‌ها و مدل‌های پایه*
- *گام‌ها*:
  1. جمع‌آوری *نمونه‌داده‌های تاریخی* (CSV/Excel از سنسورهای حفاری)  
  2. پیش‌پردازش داده‌ها با Pandas (پاک‌سازی نویز، نرمال‌سازی)  
  3. آموزش *مدل XGBoost* برای پیش‌بینی تلفات سیال (کد نمونه در بخش ۳)  
  4. ذخیره‌سازی نتایج در PostgreSQL  

- *خروجی*:  
  - یک مدل ML با دقت >۸۵% روی داده‌های تست  

### *📅 هفته ۲: شبیه‌سازی ۳D ساده*
- *گام‌ها*:
  1. مدل‌سازی هندسی سازند در *Blender* یا *Unity3D*  
  2. اعتنمال تنش‌های مکانیکی با *Python (FEniCS برای FEM)*  
  3. نمایش تعاملی تنش‌های بحرانی در محیط ۳D  

- *خروجی*:  
  - یک فایل اجرایی شبیه‌سازی با قابلیت نمایش مناطق پرخطر  

### *📅 هفته ۳: توسعه داشبورد مدیریتی*
- *گام‌ها*:
  1. راه‌اندازی *Backend* با FastAPI (خواندن داده از DB)  
  2. طراحی *فرانت‌اند* با React.js + Plotly برای نمودارها  
  3. اتصال به مدل ML برای پیش‌بینی بلادرنگ  

- *خروجی*:  
  - یک URL محلی (مثل localhost:3000) برای نمایش داده‌ها و هشدارها  

### *📅 هفته ۴: تست یکپارچه و مستندات*
- *گام‌ها*:
  1. ادغام تمام کامپوننت‌ها  
  2. تست عملکرد با *داده‌های واقعی*  
  3. تهیه گزارش فنی از نتایج  

- *خروجی*:  
  - یک *مستند PDF* شامل دقت مدل‌ها، محدودیت‌ها و پیشنهادات  

---

## *۳. نمونه کد (ادامه‌ی PoC)*
### *۳.۱. شبیه‌سازی تنش با FEniCS (پایتون)*
```python
from fenics import *

# ایجاد شبکه‌ی هندسی (مثال ساده)
mesh = UnitCubeMesh(10, 10, 10)
V = VectorFunctionSpace(mesh, 'P', 1)

# تعریف شرایط مرزی
def boundary(x):
    return x[0] < 0.5  # نیمی از سازند ثابت است

bc = DirichletBC(V, Constant((0, 0, 0)), boundary)

# محاسبه تنش‌ها
u = TrialFunction(V)
v = TestFunction(V)
f = Constant((0, 0, -1))  # نیروی وارده (مثال: فشار سیال)
a = inner(grad(u), grad(v)) * dx
L = inner(f, v) * dx

u = Function(V)
solve(a == L, u, bc)

# ذخیره‌سازی برای نمایش در Paraview/Blender
File("stress_simulation.pvd") << u


### *۳.۲. رابط FastAPI برای داشبورد*
python
from fastapi import FastAPI
import pandas as pd
import xgboost as xgb

app = FastAPI()
model = xgb.XGBClassifier()
model.load_model("fluid_loss_model.json")

@app.get("/predict")
async def predict(pressure: float, flow_rate: float):
    input_data = [[pressure, flow_rate, 30, 1500]]  # مقادیر فرضی
    risk = model.predict(input_data)[0]
    return {"risk_level": str(risk)}
```


---

## *۴. سخت‌افزار مورد نیاز برای PoC*
- *حداقل منابع*:
  - *پردازش: لپ‌تاپ با **CPU 4-core + 16GB RAM* (مثل Intel i7)  
  - *گرافیک: GPU با **۴GB VRAM* (مثل NVIDIA GTX 1650) برای شبیه‌سازی سبک  
  - *ذخیره‌سازی*: SSD 256GB  

- *توصیه برای دقت بالاتر*:
  - استفاده از *Google Colab Pro* (دسترسی به GPU Tesla T4)  
  - یا سرور ابری با *GPU NVIDIA T4* (هزینه ~$0.35/hour در AWS)  

---

## *۵. معیارهای ارزیابی موفقیت PoC*
| *معیار* | *هدف* | *ابزار اندازه‌گیری* |
|-----------|---------|----------------------|
| دقت مدل XGBoost | >۸۵% | sklearn.metrics |
| تأخیر پردازش داده | <۳ ثانیه | Python time module |
| نرخ فریم شبیه‌سازی ۳D | >۳۰ FPS | Unity Profiler |
| تعداد کاربران همزمان داشبورد | ۵+ کاربر | Locust تست بار |

---


# *MVP (حداقل محصول قابل عرضه) - سیستم پایش آسیب سازند*

## *۱. ویژگی‌های کلیدی MVP*
این نسخه حداقلی بر روی هسته اصلی سیستم تمرکز دارد تا امکان اعتبارسنجی فنی و بازخورد اولیه از کاربران را فراهم کند:

### *۱.۱. ماژول‌های اصلی*
| ماژول | فناوری‌های استفاده شده | توضیحات |
|-------|----------------------|---------|
| *پیش‌بینی تلفات سیال* | XGBoost + FastAPI | مدل پایه برای پیش‌بینی ریسک افتادگی سیال |
| *شبیه‌سازی تنش ۲D* | Python (Matplotlib) | نمایش ساده توزیع تنش در مقطع سازند |
| *داشبورد مدیریتی* | Streamlit (Python) | نمایش داده‌ها و هشدارها در یک رابط ساده |
| *ذخیره‌سازی داده‌ها* | SQLite | پایگاه داده سبک برای ذخیره نتایج |

### *۱.۲. محدودیت‌های نسخه MVP*
- شبیه‌سازی فقط در *۲ بعد* انجام می‌شود (به جای ۳D)
- مدل‌های ML تنها روی *۴ پارامتر اصلی* آموزش می‌بینند
- پردازش داده به صورت *غیربلادرنگ* (Batch Processing)

---

## *۲. معماری فنی MVP*
mermaid
flowchart TD
    A[داده‌های سنسورها CSV] --> B[پیش‌پردازش با Pandas]
    B --> C[مدل XGBoost]
    C --> D[ذخیره نتایج در SQLite]
    D --> E[داشبورد Streamlit]
    C --> F[شبیه‌سازی تنش ۲D]


## *۳. کدهای اصلی MVP*

### *۳.۱. مدل پیش‌بینی (Python)*
```python
import pandas as pd
import xgboost as xgb
from sklearn.model_selection import train_test_split

# بارگذاری داده‌های نمونه
data = pd.read_csv("well_data.csv")
features = data[["pressure", "flow_rate", "depth", "viscosity"]]
target = data["fluid_loss_risk"]

# آموزش مدل
model = xgb.XGBClassifier(n_estimators=50, max_depth=3)
model.fit(features, target)

# ذخیره مدل
model.save_model("mvp_model.json")
```

### *۳.۲. شبیه‌سازی تنش ۲D (Python)*
```python
import numpy as np
import matplotlib.pyplot as plt

def plot_stress(pressure):
    x = np.linspace(0, 10, 100)
    y = np.linspace(0, 5, 50)
    X, Y = np.meshgrid(x, y)
    stress = pressure * np.exp(-Y)  # محاسبه ساده تنش
    
    plt.contourf(X, Y, stress, levels=20)
    plt.colorbar()
    plt.title("توزیع تنش در سازند")
    plt.savefig("stress_plot.png")
```


### *۳.۳. داشبورد (Streamlit)*
```python
import streamlit as st
import pandas as pd
import xgboost as xgb

# بارگذاری مدل
model = xgb.XGBClassifier()
model.load_model("mvp_model.json")

st.title("داشبورد پایش سازند")

# ورودی کاربر
pressure = st.slider("فشار (psi)", 0, 5000, 2500)
flow_rate = st.slider("دبی جریان (gpm)", 0, 500, 250)

# پیش‌بینی
if st.button("محاسبه ریسک"):
    input_data = [[pressure, flow_rate, 1500, 30]]
    risk = model.predict(input_data)[0]
    st.success(f"سطح ریسک: {risk} (0=کم, 1=زیاد)")
    
    # نمایش شبیه‌سازی
    st.image("stress_plot.png")
```

---

## *۴. راه‌اندازی MVP*

### *۴.۱. پیش‌نیازها*
- Python 3.8+
- کتابخانه‌های مورد نیاز:
  ```bash
  pip install xgboost pandas streamlit matplotlib scikit-learn
  ```

### *۴.۲. دستورات اجرا*
۱. آموزش مدل:
   ```bash
   python train_model.py
   ```
   
۲. اجرای داشبورد:
   ```bash
   streamlit run dashboard.py
   ```

---

## *۵. منابع سخت‌افزاری مورد نیاز*
| کامپوننت | حداقل نیازمندی |
|----------|----------------|
| CPU | 4-core (Intel i5 یا معادل) |
| RAM | 8GB |
| فضای ذخیره‌سازی | 10GB (برای داده‌های نمونه) |
| سیستم عامل | Windows/Linux/macOS |

---

## *۶. معیارهای ارزیابی موفقیت MVP*
| معیار | هدف | روش اندازه‌گیری |
|-------|-----|----------------|
| دقت پیش‌بینی | >80% | تست دقت مدل |
| زمان پاسخگویی | <5 ثانیه | زمان بارگذاری داشبورد |
| قابلیت نمایش داده | ۵ نوع نمودار | بررسی رابط کاربری |
| پایداری سیستم | ۲۴ ساعت کار مداوم | مانیتورینگ منابع |

---

## *۷. گام‌های بعدی پس از MVP*
- اضافه کردن *شبیه‌سازی ۳D* با Unity
- پیاده‌سازی *پردازش بلادرنگ* داده‌ها
- توسعه *مدل‌های ترکیبی* (LSTM + XGBoost)
- یکپارچه‌سازی با *سیستم‌های SCADA* موجود

این MVP در *۲ هفته* قابل توسعه است و نیاز به داده‌های نمونه از سنسورهای واقعی دارد. که در ادامه برای شما ارسال می شود


