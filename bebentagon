import streamlit as st
import google.generativeai as genai
from PIL import Image

# --- 設定（APIキー） ---
API_KEY = "AIzaSyDpANPLV-y_RRY04jTatHLqqUnbrNv3gs0"
genai.configure(api_key=API_KEY)

# --- デザイン設定（淡いブルー & 明朝体） ---
st.set_page_config(page_title="簡単旅note", layout="centered")

st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Shippori+Mincho&display=swap');
    
    /* 背景色を淡いブルーに */
    .stApp {
        background-color: #E0F2F7;
    }
    
    /* 全体のフォントを明朝体に */
    html, body, [class*="css"], .stMarkdown, p, h1, h2, h3 {
        font-family: 'Shippori Mincho', serif !important;
        color: #37474F;
    }
    
    /* 入力フォームの角を丸く */
    .stTextInput>div>div>input, .stTextArea>div>div>textarea {
        border-radius: 15px;
        background-color: #FFFFFF;
    }
    
    /* ボタンを柔らかいデザインに */
    .stButton>button {
        border-radius: 25px;
        background-color: #4DD0E1;
        color: white;
        border: none;
        padding: 0.5rem 2rem;
        transition: 0.3s;
    }
    .stButton>button:hover {
        background-color: #26C6DA;
        transform: scale(1.05);
    }
    </style>
    """, unsafe_allow_html=True)

# --- アプリ画面 ---
st.title("🌿 簡単旅note")
st.caption("バックパッカーのための、エモくて売れる旅ログ作成機")

# サイドバー：要望入力
with st.sidebar:
    st.header("🧳 旅の記録")
    place = st.text_input("訪れた場所", placeholder="例：イスタンブールの旧市街")
    price = st.text_input("予算・価格", placeholder="例：サバサンド 100リラ")
    vibe = st.selectbox("記事の雰囲気", ["エモい（叙情的）", "泥臭い（リアル）", "淡々と（記録）", "ワクワク（観光ガイド風）"])
    word_count = st.select_slider("目標文字数", options=[300, 500, 800, 1200, 1500], value=800)
    
    st.divider()
    core_point = st.text_area("話の核（有料ポイント）", placeholder="ここだけにしかない秘密や苦労話")
    message = st.text_area("伝えたいこと", placeholder="読者に最後に言いたい一言")

# メイン画面：写真アップロード
uploaded_files = st.file_uploader("写真をアップロード（最大10枚）", type=['jpg', 'jpeg', 'png'], accept_multiple_files=True)

if uploaded_files:
    # 選択した写真が10枚を超えないようにチェック
    if len(uploaded_files) > 10:
        st.error("写真は10枚以内にしてください。")
    else:
        cols = st.columns(5)
        for i, file in enumerate(uploaded_files):
            with cols[i % 5]:
                st.image(file, use_container_width=True)

# 生成ボタン
if st.button("noteの下書きを紡ぐ"):
    if not uploaded_files:
        st.warning("写真を1枚以上アップロードしてください。")
    else:
        model = genai.GenerativeModel('gemini-1.5-flash')
        images = [Image.open(f) for f in uploaded_files]
        
        prompt = f"""
        あなたはプロの旅ライターです。提供された写真と以下の情報から、noteに投稿するためのエモい記事を作成してください。
        
        【情報】
        場所: {place}
        価格・予算感: {price}
        雰囲気: {vibe}
        文字数目安: {word_count}字
        話の核（有料部分にしたい内容）: {core_point}
        伝えたいメッセージ: {message}
        
        【構成ルール】
        1. 読者がクリックしたくなるエモいタイトルを付ける。
        2. 無料部分では、現地の匂いや音、感情を大切にした文章を書く。
        3. 有料部分の直前に「ここから先は、実際に足を運んだ人だけが知る特別な情報です」といった魅力的な引き文を入れる。
        4. 有料部分として指定された『話の核』を詳しく展開する。
        5. 全体を通して『明朝体』が似合う、バックパッカーらしい少し落ち着いたトーンで書く。
        """
        
        with st.spinner("思い出を言葉にしています..."):
            response = model.generate_content([prompt] + images)
            st.session_state.generated_text = response.text

# 結果の表示と追加修正
if 'generated_text' in st.session_state:
    st.divider()
    st.subheader("📝 生成された下書き")
    
    # 自分で編集できるエリア
    edited_text = st.text_area("自由に編集してください", value=st.session_state.generated_text, height=500)
    
    # AIへの追加要望
    feedback = st.text_input("AIにさらに追加でお願いする", placeholder="例：もう少し安さを強調して、最後をエモく締めて")
    if st.button("追加修正を依頼"):
        model = genai.GenerativeModel('gemini-1.5-flash')
        retry_prompt = f"以下の文章を、次の要望に合わせて修正してください：{feedback}\n\n元文章：\n{edited_text}"
        with st.spinner("修正中..."):
            response = model.generate_content(retry_prompt)
            st.session_state.generated_text = response.text
            st.rerun()

    st.success("完成しました！文章をコピーしてnoteに投稿しましょう。")
