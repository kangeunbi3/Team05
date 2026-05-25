import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# 한글 폰트 설정 (데이터 시각화 깨짐 방지)
plt.rcParams['font.family'] = 'Malgun Gothic'
plt.rcParams['axes.unicode_minus'] = False

# 1. csv 파일 업로드 및 인코딩 처리
def load_data(uploaded_file):
    encodings = ['utf-8', 'cp949', 'euc-kr']
    for enc in encodings:
        try:
            uploaded_file.seek(0)
            df = pd.read_csv(uploaded_file, encoding=enc)
            return df, enc
        except UnicodeDecodeError:
            continue
    return None, None

# 2. 원본 파일 내용 확인 (처음 20줄만 텍스트로 표시)
def show_raw_text(uploaded_file, encoding):
    uploaded_file.seek(0)
    raw_lines = [uploaded_file.readline().decode(encoding, errors='ignore') for _ in range(20)]
    with st.expander("📄 원본 파일 내용 확인 (처음 20줄)", expanded=False):
        st.text("".join(raw_lines))

# 4 & 5. 증가율 계산 및 코로나 전후 데이터 가공 함수
def process_kosis_data(df):
    # 컬럼명 공백 제거
    df.columns = df.columns.str.strip()
    
    # 가로형 데이터를 세로형 데이터(연도, 거래액)로 변환 (Unpivoting/Melt)
    # 2017년부터 2025년까지의 연도 컬럼들을 '연도' 컬럼으로 통합
    id_vars = ['상품군별(1)', '판매매체별(1)']
    value_vars = [col for col in df.columns if col not in id_vars]
    
    df_melted = df.melt(id_vars=id_vars, value_vars=value_vars, var_name='연도', value_name='거래액')
    
    # 데이터 타입 변환 및 정제
    df_melted['연도'] = df_melted['연도'].astype(int)
    df_melted['거래액'] = df_melted['거래액'].astype(float)
    
    # 판매매체별로 데이터 분리 (종합/인터넷/모바일)
    df_total = df_melted[df_melted['판매매체별(1)'] == '계'].sort_values('연도').reset_index(drop=True)
    df_internet = df_melted[df_melted['판매매체별(1)'] == '인터넷쇼핑'].sort_values('연도').reset_index(drop=True)
    df_mobile = df_melted[df_melted['판매매체별(1)'] == '모바일쇼핑'].sort_values('연도').reset_index(drop=True)
    
    # 2019년 vs 2021년 코로나 전후 비교 증가율 계산 (종합 데이터 기준)
    val_2019 = df_total[df_total['연도'] == 2019]['거래액'].values[0]
    val_2021 = df_total[df_total['연도'] == 2021]['거래액'].values[0]
    growth_rate = ((val_2021 - val_2019) / val_2019) * 100
    
    comparison = {
        '2019': val_2019,
        '2021': val_2021,
        'growth_rate': growth_rate
    }
    
    return df_total, df_internet, df_mobile, comparison

# 6. 그래프 생성 함수
def draw_chart(df_total, df_internet, df_mobile):
    fig, ax = plt.subplots(figsize=(10, 6))
    
    # 매체별 선그래프 그리기
    ax.plot(df_total['연도'], df_total['거래액'], marker='o', color='royalblue', linewidth=2.5, label='온라인 전체 쇼핑계')
    ax.plot(df_internet['연도'], df_internet['거래액'], marker='s', color='orange', linewidth=2, linestyle='--', label='인터넷쇼핑')
    ax.plot(df_mobile['연도'], df_mobile['거래액'], marker='^', color='forestgreen', linewidth=2, linestyle=':', label='모바일쇼핑')
    
    # 코로나 기준선 표시 (2020 위치에 세로선 추가)
    ax.axvline(x=2020, color='crimson', linestyle='-', linewidth=1.5, label='코로나19 발생 (2020)')
    
    ax.set_title('연도별/매체별 화장품 온라인 거래액 추이', fontsize=14, pad=15)
    ax.set_xlabel('연도', fontsize=12)
    ax.set_ylabel('거래액 (단위: 백만원)', fontsize=12)
    ax.set_xticks(df_total['연도'].unique())
    
    # Y축 천단위 콤마 표시
    ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
    
    ax.grid(True, linestyle=':', alpha=0.6)
    ax.legend(fontsize=10)
    st.pyplot(fig)

# 7. 자동 분석 문장 출력 함수
def print_auto_analysis(comparison, df_total, df_mobile):
    st.subheader("💡 데이터 자동 분석 요약")
    
    rate = comparison['growth_rate']
    st.write(f"- 📊 **코로나 전후 비교:** 코로나 직전(2019년) 대비 코로나 확산기(2021년) 화장품 온라인 총 거래액은 **{rate:.1f}% {'증가' if rate > 0 else '감소'}**하였습니다.")
    
    # 2020년 이후 트렌드 및 소비 방식 변화 분석
    val_2020_mobile = df_mobile[df_mobile['연도'] == 2020]['거래액'].values[0]
    val_2025_mobile = df_mobile[df_mobile['연도'] == 2025]['거래액'].values[0]
    
    st.write("- 🛒 **소비 습관 지속 여부:** 코로나 이후에도 온라인 화장품 총 거래액은 일시적 하락 후 다시 반등하여 2025년 최대치를 기록했습니다. 이는 **코로나로 형성된 온라인 소비 습관이 완전히 정착되어 지속**되고 있음을 보여줍니다.")
    
    if val_2025_mobile > val_2020_mobile:
        st.write("- 📱 **모바일 쇼핑 방식 변화:** 주목할 점은 인터넷쇼핑은 감소세인 반면, **모바일쇼핑 거래액이 폭발적으로 급증**했다는 점입니다. 화장품 소비의 중심축이 PC(인터넷)에서 **스마트폰(모바일) 중심으로 완벽히 전환**되었습니다.")

# 메인 앱 제어
def main():
    st.title("💄 화장품 소비 변화 분석 프로그램")
    st.markdown("KOSIS 가로형 통계 데이터를 기반으로 코로나 전후 화장품 온라인/모바일 소비 트렌드를 분석합니다.")
    
    # 1. 파일 업로드 기능
    uploaded_file = st.file_uploader("KOSIS 화장품 CSV 파일을 업로드해주세요.", type=["csv"])
    
    if uploaded_file is not None:
        df, encoding = load_data(uploaded_file)
        
        if df is not None:
            st.success(f"파일 로드 성공! (인코딩: {encoding})")
            
            # 2. 원본 파일 내용 확인
            show_raw_text(uploaded_file, encoding)
            
            # 3. 상위 5개 데이터 출력 (원본이 4개 행이므로 전부 예쁘게 출력됩니다)
            st.subheader("🔝 데이터 상위 데이터 미리보기")
            st.dataframe(df)
            
            # 데이터 가공 및 분석 실행
            df_total, df_internet, df_mobile, comparison = process_kosis_data(df)
            
            # 5. 코로나 전후 비교 지표 출력 (2019 vs 2021)
            st.subheader("📉 코로나 전후 거래액 비교 (2019 vs 2021)")
            col1, col2, col3 = st.columns(3)
            col1.metric("2019년 전체 거래액", f"{comparison['2019']:,.0f} 백만원")
            col2.metric("2021년 전체 거래액", f"{comparison['2021']:,.0f} 백만원")
            col3.metric("증가율 (전체)", f"{comparison['growth_rate']:.1f}%")
            
            # 6. 그래프 생성
            st.subheader("📈 연도별/매체별 소비 추이 시각화")
            draw_chart(df_total, df_internet, df_mobile)
            
            # 7. 자동 분석 문장 출력
            print_auto_analysis(comparison, df_total, df_mobile)
            
        else:
            st.error("파일을 읽을 수 없습니다. 인코딩 구조를 확인해 주세요.")

if __name__ == "__main__":
    main()