import streamlit as st
import pandas as pd
import json

# Sidebar: configuration
st.sidebar.header("Settings")

# Main content
st.header('''
          เครื่องมือสนับสนุนการ migrate จาก LINE Notify สู่ LINE Official Account
          โดย ผศ.ดร.ศุภชัย วรพจน์พิศุทธิ์ และสมาคมสมองกลฝังตัวไทย
          ''')
#profile = st.sidebar.text_input("โปรไฟล์เดิม", value='no profile')
num_devices = st.sidebar.number_input("จำนวนอุปกรณ์", min_value=1, value=1, key='num_devices')
num_users = st.sidebar.number_input("จำนวนผู้ใช้งาน", min_value=1, value=1, key='num_users')

# if profile != 'no profile':
#     print('Importing profile')
#     try:
#         iot_devices = json.load(profile+'_devices.json')
#     except:
#         print('No profile, use empty data')
#         iot_devices = {}
# else:
#     iot_devices = {}
iot_devices = {}

info_tab, data_tab, calc_tab, sol_tab = st.tabs(["วิธีใช้งาน", "ข้อมูลระบบ", "คำนวณค่าใช้จ่าย", "แนะนำการ migrate"])

#1. หากมีข้อมูลเดิม หรือต้องการบันทึกเพื่อใช้ภายหลัง ให้กรอก **keyword** ลงในช่อง **โปรไฟล์เดิม**
with info_tab:
    st.write('''
    #### วิธีใช้งาน    
    1. ป้อนข้อมูลจำนวนอุปกรณ์ โดยพิจารณาจาก
        - อุปกรณ์ N ตัว ติดตั้งในสถานที่เดียวกัน ให้ป้อนค่า N ตัว
        - อุปกรณ์ N ตัว ติดตั้งใน M สถานที่ มีการรับข้อมูลซ้ำกัน ให้ป้อนค่า N x M 
        - อุปกรณ์ N ตัว ติดตั้งใน M สถานที่ แต่ละสถานที่ไม่รับข้อความซ้ำกัน ให้ป้อนค่า N แล้วแยกคำนวณแต่ละสถานที่
    2. ป้อนข้อมูลจำนวนผู้ใช้งาน
        - ป้อนจำนวนผู้ใช้งาน หากทุกอุปกรณ์ส่งข้อความไปยังกลุ่มผู้ใช้งานเดียวกัน 
        - ป้อนค่า 0 หากแต่ละอุปกรณ์ส่งข้อความไปยังกลุ่มผู้ใช้งานที่แตกต่างกัน จากนั้นไปกรอกข้อมูลจำนวนผู้ใช้งานแต่ละกลุ่มย่อยใน tab **ป้อนข้อมูล**
    3. เลือก tab **ข้อมูลระบบ** จากนั้นป้อนข้อมูลความถี่ในการแจ้งเตือน/รายงาน
    4. เลือก tab **คำนวณค่าใช้จ่าย** เพื่อตรวจสอบค่าบริการ
    5. เลือก tab **แนะนำการ migrate** เพื่อดูวิธีการ migrate จาก LINE Notify สู่ LINE Official Account
    ''')

    st.write('''
    #### ข้อมูลบริการ LINE Notify และ LINE Official Account
             
    ##### การส่งข้อความจากอุปกรณ์ IoT ผ่าน LINE Notify 
    การใช้บริการเริ่มจากลงทะเบียนเข้าใช้งาน LINE Notify จากนั้นเตรียม 
    - QR Code สำหรับให้ผู้ใช้สมัครรับข้อความ
    - token สำหรับใช้ส่งข้อความผ่าน REST API
             
    เมื่ออุปกรณ์ IoT ส่ง POST request ด้วย token ไปยังบริการ LINE Notify ทาง LINE จะส่งข้อความไปยังผู้ใช้งานที่ลงทะเบียนไว้แล้ว
    การสื่อสารจะเป็นการส่งข้อมูลแบบทางเดียว เน้นแจ้งเตือน/รายงานสถานะของอุปกรณ์ **ไม่มีค่าบริการในการส่งข้อความ**

    ##### การส่งข้อความจากอุปกรณ์ IoT ผ่าน LINE Official Account
    การส่งข้อความต้องเตรียม server สำหรับเชื่อมต่อ LINE messaging API และ database สำหรับเก็บข้อมูลของผู้ใช้งาน การพัฒนาบริการเริ่มจาก
    - สมัคร LINE Official Account
    - สร้าง LINE messaging API channel จะได้ QR code สำหรับให้ผู้ใช้งานสมัคร หรืออาจสมัครผ่านการ search
    - นำ secret และ access token ไปเขียนโค้ด webhook 
    - สร้าง database สำหรับเก็บข้อมูล user id ของผู้ใช้งานตอนสมัคร (ผ่าน follow event)
    - เตรียม REST endpoint สำหรับรับข้อมูลจากอุปกรณ์ IoT
    
    **มีค่าบริการในการส่งข้อความ** โดยนับจากจำนวนผู้ใช้งานที่ได้รับข้อความ (reach)
    - free plan ส่งได้ไม่เกิน 300 reach/เดือน
    - basic plan เดือนละ 1,280+VAT บาท คิดเงินเพิ่มหากเกิน 15,000 reach/เดือน
    - pro plan เดือนละ 1,780+VAT บาท คิดเงินเพิ่มหากเกิน 35,000 reach/เดือน 

    การเตรียม server จะมีค่าบริการในส่วน server และต้องพัฒนาส่วนของระบบ token เอง
    ในกรณีต้องการสื่อสารระหว่างอุปกรณ์ IoT และผู้ใช้งานแบบสองทาง แนะนำให้ใช้โพรโทคอล MQTT ในการรับส่งข้อมูล ซึ่งจะมีค่าใช้จ่ายเพิ่มในส่วน private MQTT broker 
    ''')

with data_tab:
    st.write("#### ข้อมูลอุปกรณ์ IoT")
    if len(iot_devices) == 0:
        iot_devices = {}
        for i in range(num_devices):
            iot_devices[i] = {'name':'dev_'+str(i), 
                              'num_users':1, 
                              'mode':'แจ้งเตือน', 
                              'frequency':'1 ครั้ง/วัน',
                              'over_limit': False}
            device_df = pd.DataFrame.from_dict(iot_devices, orient='index')
    if num_users > 0:
        for i in range(num_devices):
            device_df.loc[i, 'num_users'] = num_users
    col_cfg = {
        '_index': None,
        'name': st.column_config.TextColumn('อุปกรณ์'),
        'num_users': st.column_config.NumberColumn('จำนวนผู้ใช้งาน', format='%d คน'),
        'mode': st.column_config.SelectboxColumn('โหมด', options=['แจ้งเตือน', 'รายงาน']),
        'frequency': st.column_config.SelectboxColumn('โอกาสเหตุการณ์/ความถี่รายงาน', options=['1 ครั้ง/วัน', '2 ครั้ง/วัน', '1 ครั้ง/สัปดาห์', '1 ครั้ง/เดือน', 'ทุก 15 นาที', 'ทุก 30 นาที', 'ทุก 1 ชั่วโมง']),
        'over_limit': st.column_config.CheckboxColumn('ข้อความ > 5 kB')
    }
    device_df = st.data_editor(device_df, use_container_width=True ,column_config=col_cfg, key='device_df')
    
with calc_tab:
    lineoa_package = st.selectbox('แพ็คเกจ', options=['Free plan', 'Basic plan', 'Pro plan'], index=1, key='lineoa_package')

    # copy คอลัมน์ name num_devices และ num_users ของ device_df มาใส่ cost_df
    cost_df = device_df[['name', 'mode', 'num_users']]
    num_reach = []
    msg_cost = []
    num_msgs = 0
    for row in device_df.iterrows():
        user_count = row[1].loc['num_users']
        msg_freq = row[1].loc['frequency'] 
        redundant = 2 if row[1].loc['over_limit'] else 1 
        if msg_freq == '1 ครั้ง/วัน':
            num_reach.append(user_count*1*30 * redundant)
            num_msgs += 1*30 * redundant
        elif msg_freq == '2 ครั้ง/วัน':
            num_reach.append(user_count*2*30 * redundant)
            num_msgs += 2*30 * redundant
        elif msg_freq == '1 ครั้ง/สัปดาห์':
            num_reach.append(user_count*1*4 * redundant)
            num_msgs += 1*4 * redundant
        elif msg_freq == '1 ครั้ง/เดือน':
            num_reach.append(user_count*1 * redundant)
            num_msgs += 1 * redundant
        elif msg_freq == 'ทุก 15 นาที': 
            num_reach.append(user_count*4*24*30 * redundant)
            num_msgs += 4*24*30 * redundant
        elif msg_freq == 'ทุก 30 นาที':
            num_reach.append(user_count*2*24*30 * redundant)
            num_msgs += 2*24*30 * redundant
        else: #'ทุก 1 ชั่วโมง'
            num_reach.append(user_count*24*30 * redundant)
            num_msgs += 24*30 * redundant
        
    cost_df = cost_df.assign(num_reach=num_reach)
    if lineoa_package == 'Basic plan':
        cost_df['msg_cost'] = cost_df['num_reach'] * 0.10
    elif lineoa_package == 'Pro plan':
        cost_df['msg_cost'] = cost_df['num_reach'] * 0.06
    else:
        cost_df['msg_cost'] = cost_df['num_reach'] * 0.0

    col_cfg = {
        '_index': None,
        'name': st.column_config.TextColumn('อุปกรณ์'),
        'mode': st.column_config.SelectboxColumn('โหมด', options=['แจ้งเตือน', 'รายงาน']),
        'num_users': st.column_config.NumberColumn('จำนวนผู้ใช้งาน', format='%d คน'),
        'num_reach': st.column_config.NumberColumn('จำนวน reach/เดือน', format='%d ครั้ง'),
        'msg_cost': st.column_config.NumberColumn('มูลค่าของ reach', format='%.2f บาท')
    }
    cost_df.sort_values('num_reach', ascending=False, inplace=True)
    st.dataframe(cost_df, use_container_width=True, column_config=col_cfg)
    total_reach = cost_df['num_reach'].sum()
    if lineoa_package == 'Basic plan':
        monthly_cost = 1280.0
        over_reach = total_reach - 15000
        if over_reach > 0:
            over_reach_cost = over_reach * 0.10
        else:
            over_reach_cost = 0.0
        lineoa_cost = (monthly_cost + over_reach_cost) * 1.07
        st.write(f'''
        ##### ค่าใช้จ่าย LINE Official Account (Basic Plan)
        จำนวนข้อความทั้งหมด {total_reach} ครั้ง
        ค่าบริการรายเดือน {monthly_cost:.2f} บาท + ค่าบริการข้อความเกินโควต้า {over_reach_cost:.2f} บาท รวมเป็นค่าใช้จ่ายรวม VAT เท่ากับ {lineoa_cost:.2f} บาท         
        ''')

        st.write('##### ปัจจัยที่ส่งผลต่อค่าใช้จ่าย')
        cost_factors = {
            'ค่าบริการรายเดือน': (monthly_cost*1.07) /  lineoa_cost * 100,
        }
        cost_adj_df = cost_df.copy()
        if total_reach > 15000:
            adj_scale = (total_reach - 15000) / total_reach
        else:
            adj_scale = 1.0
        cost_adj_df['msg_cost'] = cost_adj_df['msg_cost'] * adj_scale
        for device in cost_adj_df.head(3).iterrows():
            cost_factors[device[1][0]] = (device[1][4] * 1.07) / lineoa_cost * 100
        st.bar_chart(pd.Series(cost_factors), use_container_width=True)

    elif lineoa_package == 'Pro plan':
        monthly_cost = 1780.0
        over_reach = total_reach - 35000
        if over_reach > 0:
            over_reach_cost = over_reach * 0.06
        else:
            over_reach_cost = 0.0
        lineoa_cost = (monthly_cost + over_reach_cost) * 1.07
        st.write(f'''
        ##### ค่าใช้จ่าย LINE Official Account (Pro Plan)
        จำนวนข้อความทั้งหมด {total_reach} ครั้ง
        ค่าบริการรายเดือน {monthly_cost:.2f} บาท + ค่าบริการข้อความเกินโควต้า {over_reach_cost:.2f} บาท รวมเป็นค่าใช้จ่ายรวม VAT เท่ากับ {lineoa_cost:.2f} บาท         
        ''')

        st.write('##### ปัจจัยที่ส่งผลต่อค่าใช้จ่าย')
        cost_factors = {
            'ค่าบริการรายเดือน': (monthly_cost*1.07) /  lineoa_cost * 100,
        }
        cost_adj_df = cost_df.copy()
        if total_reach > 35000:
            adj_scale = (total_reach - 35000) / total_reach
        else:
            adj_scale = 1.0
        cost_adj_df['msg_cost'] = cost_adj_df['msg_cost'] * adj_scale
        for device in cost_adj_df.head(3).iterrows():
            cost_factors[device[1][0]] = (device[1][4] * 1.07) / lineoa_cost * 100
        st.bar_chart(pd.Series(cost_factors), use_container_width=True)

    else:
        if total_reach > 300:
            st.write(f'''
            ##### บริการ Free plan
            จำนวนข้อความทั้งหมด {total_reach} ครั้ง
            ไม่สามารถใช้ Free plan ได้ เพราะเกินโควต้า 300 reach/เดือน
            ''')
        else:
            st.write('''
            ##### บริการ Free plan
            ไม่มีค่าใช้จ่าย เพราะไม่เกินโควต้า 300 reach/เดือน
            ''')
        lineoa_cost = 0
    st.session_state['lineoa_cost'] = lineoa_cost         

    st.write('''
    #### การคำนวณ reach และเลือกแพ็คเกจ
    การคำนวณค่าใช้จ่ายจะดูจากจำนวน reach = จำนวนข้อความ/เดือน * จำนวนผู้ติดตาม (ไม่บล็อก/ใช้งานอยู่) จากนั้นหักจำนวนครั้งในการส่งฟรีของแพ็คเกจ แล้วจึงคูณด้วยค่าใช้จ่าย/reach
    - basic plan เดือนละ 1,280 บาท ส่งฟรีได้ 15,000 reach และส่งเกิน 0.10 บาท/reach 
    - pro plan เดือนละ 1,780 บาท ส่งฟรีได้ 15,000 reach และส่งเกิน 0.06 บาท/reach

    **ค่าใช้จ่ายทั้งหมดจะต้องถูกนำมาคิด VAT ด้วย**             
    ''')

with sol_tab:
    st.write('''
    #### แบบสอบถามเกี่ยวกับบริการทดแทน       
    ''')
    user_app = st.selectbox("แอพของผู้ใช้", ["LINE app เท่านั้น", "แอพอื่นบนสมาร์ทโฟน/คอมพิวเตอร์"], key='user_app')
    if user_app == 'LINE app เท่านั้น':
        service_payer = st.selectbox("ผู้จ่ายค่าบริการ", ["หน่วยงานลูกค้า", "ผู้พัฒนา", "ลูกค้า/ผู้พัฒนาจ่ายร่วม"], key='service_payer')
        if service_payer != "หน่วยงานลูกค้า":
            msg_choice = {}
            msg_choice['change'] = st.selectbox("การส่งข้อมูลให้ผู้ใช้งาน", ["ไม่สามารถปรับเปลี่ยนได้", "สามารถปรับเปลี่ยนได้"], key='info_flow')
            if msg_choice['change'] == "สามารถปรับเปลี่ยนได้":
                st.write("**การจำกัดประเภทข้อมูลที่ส่ง**")
                msg_choice['emergency'] = st.checkbox("แจ้งเตือนฉุกเฉิน", value=True, key='emergency_noti')
                msg_choice['overview'] = st.checkbox("รายงานภาพรวม", value=False, key='overview_report')
                msg_choice['status'] = st.checkbox("รายงานสถานะล่าสุด", value=False, key='status_report')
            user_choice = {}
            user_choice['change'] = st.selectbox("กลุ่มผู้ใช้งาน", ["ต้องส่งถึงทุกคน", "ส่งเฉพาะผู้เกี่ยวข้อง", ], key='user_type')
            if user_choice['change'] == "ส่งเฉพาะผู้เกี่ยวข้อง":
                st.write("**กลุ่มผู้เกี่ยวข้อง**")
                user_choice["core"] = st.checkbox("ผู้รับผิดชอบหลักเท่านั้น", value=True, key='core_member')
                user_choice["group"] = st.checkbox("กลุ่มผู้รับผิดชอบ", value=False, key='group_member')
                user_choice["others"] = st.checkbox("ผู้เกี่ยวข้องอื่นๆ", value=False, key='other_member')
    else:
        app_choice = st.selectbox("แอพทางเลือกอื่น", ["Facebook Messenger", "Discord", "Telegram", "Slack", "MS Teams"], key='app_choice')
        service_payer = st.selectbox("ผู้จ่ายค่าบริการ", ["หน่วยงานลูกค้า", "ผู้พัฒนา", "ลูกค้า/ผู้พัฒนาจ่ายร่วม"], key='service_payer')


    gen_recommend = st.button("สร้างข้อแนะนำ")
    if gen_recommend:
        # recommend service based on service settings
        def gen_recommend_lineapp_customer():
            recommend_txts = []
            recommend_txts.append("- สามารถเลือกใช้ LINE Messaging API แบบเพิ่ม LINE OA เข้ากลุ่ม ซึ่งให้บริการคล้าย LINE Notify แบบเดิม โดยแต่ละข้อความที่ส่งเข้ากลุ่มจะกระจายถึงทุกคน")
            recommend_txts.append("- ควรพิจารณาเพิ่มบริการ API แบบ user (ส่วนตัว) ซึ่งสามารถเลือกข้อความแบบเจาะจงกลุ่มผู้ใช้งานเฉพาะ (narrowcast) รวมทั้งสามารถให้ข้อมูลผ่านการตอบกลับผู้ใช้งาน (replyMessage) ที่จะไม่มีค่าใช้จ่าย")
            recommend_txts.append(f"- ควรแจ้งลูกค้าเกี่ยวกับค่าใช้จ่ายในส่วน LINE OA เท่ากับ :red[{lineoa_cost:.2f} บาท] เนื่องจากค่าบริการรายเดือน {monthly_cost:.2f} บาท และการคำนวณจำนวน reach จาก จำนวนข้อความ x จำนวนผู้ใช้งานในกลุ่ม เท่ากับ {total_reach}")
            return recommend_txts

        def gen_recommend_lineapp_developer(msg_choice, user_choice):
            recommend_txts = []
            if (msg_choice['change'] == "ไม่สามารถปรับเปลี่ยนได้") and (user_choice['change'] == "ต้องส่งถึงทุกคน"):
                recommend_txts.append(f"- การเลือกส่งข้อความผ่าน LINE messaging API เข้ากลุ่ม ซึ่งจะคล้ายกับบริการ LINE Notify จะมีค่าใช้จ่ายสูงเนื่องจากการคำนวณจำนวน reach = จำนวนข้อความ x จำนวนผู้ใช้งานในกลุ่ม ซึ่งตามสถานะข้อมูลที่กรอกไว้คือ {total_reach} คิดเป็นมูลค่า :red[{over_reach_cost} บาท]")
                recommend_txts.append("- ควรพิจารณาเพิ่มบริการ API แบบ user (ส่วนตัว) ซึ่งสามารถเลือกข้อความแบบเจาะจงกลุ่มผู้ใช้งานเฉพาะ (narrowcast) รวมทั้งสามารถให้ข้อมูลผ่านการตอบกลับผู้ใช้งาน (replyMessage) ที่จะไม่มีค่าใช้จ่าย")
            else:
                recommend_txts.append("- การเลือกใช้ LINE Messaging API แบบ user (ส่วนตัว) จะสามารถปรับรูปแบบข้อความและกลุ่มผู้ใช้งานได้ รวมทั้งรองรับการตอบกลับข้อมูลตามการสั่งของผู้ใช้งานที่จะไม่มีค่าใช้จ่าย (replyMessage)")
                if msg_choice['change'] == "สามารถปรับเปลี่ยนได้":
                    recommend_txts.append("- การจำกัดประเภทข้อมูลที่ส่ง โดยเลือกส่งข้อความเฉพาะการแจ้งเตือนเฉพาะเหตุการณ์ จะช่วยลดค่าใช้จ่ายในส่วนค่าบริการ LINE OA ได้มาก โดยสามารถเลือกส่งข้อความที่มีความสำคัญน้อยลง เช่น การรายงานสถานะ ผ่านช่องทางบริการ message แบบอื่นที่ไม่มีค่าใช้จ่าย")
                    if msg_choice['overview']:
                        recommend_txts.append("- การส่งข้อความรายงานสถานะเฉพาะภาพรวมโดยมีความถี่น้อยลง เช่น วันละ 1 ครั้ง จะช่วยลดจำนวนข้อความได้มาก")
                    if msg_choice['status']:
                        recommend_txts.append(f"- การส่งข้อความรายงานล่าสุดแต่ไม่ลดความถี่ในการรายงาน เป็นปัจจัยหลักที่ทำให้ค่าใช้จ่ายเพิ่มขึ้น")
                        recommend_txts.append(f"- การลดภาระในการรายงานสถานะล่าสุดอาจทำได้ด้วยการเลือกส่งข้อความผ่านบริการ message แบบอื่นที่ไม่มีค่าใช้จ่าย หรือใช้การตอบกลับข้อมูลตามการร้องขอเท่านั้น")
                if user_choice['change'] == "ส่งเฉพาะผู้เกี่ยวข้อง":
                    recommend_txts.append(f"- การเลือกส่งข้อความเฉพาะผู้เกี่ยวข้องตามระดับของบทบาท จะช่วยลดค่าใช้จ่ายในส่วนค่าบริการ LINE OA ได้มาก")
                    if user_choice['core']:
                        recommend_txts.append(f"- การเลือกส่งข้อความเฉพาะผู้รับผิดชอบหลักเท่านั้น จะช่วยลดจำนวนข้อความได้มาก")
                    else:
                        recommend_txts.append(f"- แนะนำให้กำหนดผู้รับผิดชอบหลักในการดำเนินการเมื่อได้รับข้อความแจ้งเตือน/รายงาน ซึ่งอาจกำหนดได้มากกว่า 1 คน หรือใช้ซอฟต์แวร์ในการแจ้งเพิ่มหากผู้รับผิดชอบหลักไม่ตอบกลับภายในเวลาที่กำหนด")
                    if user_choice['group']:
                        recommend_txts.append(f"- สำหรับผู้เกี่ยวข้องที่ไม่ใช่ผู้รับผิดชอบหลัก ควรควรได้รับข้อมูลในระดับการสรุปภาพรวม หรือตรวจสอบสถานะผ่านการส่งคำสั่ง")
                    if user_choice['others']:
                        recommend_txts.append(f"- สำหรับทีมงานทั่วไปที่ไม่เกี่ยวข้องกับอุปกรณ์โดยตรง แนะนำให้ส่งข้อความผ่านช่องทางบริการ message แบบอื่นที่ไม่มีค่าใช้จ่าย หรือให้ตอบกลับข้อมูลจากการร้องขอเท่านั้น (ไม่มีค่าใช้จ่าย)")
            return recommend_txts

        def gen_recommend_other_apps(app_choice):
            recommend_txts = []
            if app_choice == "Facebook Messenger":
                recommend_txts.append("- Facebook Messenger")
            elif app_choice == "Discord":
                recommend_txts.append("- Discord")
            elif app_choice == "Telegram":
                recommend_txts.append("- Telegram")
            elif app_choice == "Slack":
                recommend_txts.append("- Slack")
            elif app_choice == "MS Teams":
                recommend_txts.append("- MS Teams")
            else:
                recommend_txts.append("- ")
            return recommend_txts

        serv_recommend = {
            ("LINE app เท่านั้น", "หน่วยงานลูกค้า"): lambda : gen_recommend_lineapp_customer(),
            ("LINE app เท่านั้น", "ผู้พัฒนา"): lambda : gen_recommend_lineapp_developer(msg_choice, user_choice),
            ("LINE app เท่านั้น", "ลูกค้า/ผู้พัฒนาจ่ายร่วม"): lambda : gen_recommend_lineapp_developer(msg_choice, user_choice),
            ("แอพอื่นบนสมาร์ทโฟน/คอมพิวเตอร์", "หน่วยงานลูกค้า"): lambda : gen_recommend_other_apps(app_choice),
        }
        recommend_txts = serv_recommend[(user_app, service_payer)]()
        st.markdown("#### ข้อแนะนำของบริการทดแทน")
        for txt in recommend_txts:
            st.markdown(txt)

        # generate cost information based on service settings
        def summarize_cost_lineapp_customer():
            cost_txts = []
            cost_txts.append(f"- ค่าบริการ LINE OA รายเดือน {monthly_cost:.2f} บาท และค่าบริการข้อความเกินโควต้า {over_reach_cost:.2f} บาท รวมเป็นค่าใช้จ่ายรวม VAT เท่ากับ {lineoa_cost:.2f} บาท")
            cost_txts.append(f"- ค่าบริการ cloud สำหรับเซิร์ฟเวอร์ บาท")
            cost_txts.append(f"- ค่าพัฒนาซอฟต์แวร์คาดว่า บาท")
            return cost_txts

        def summarize_cost_lineapp_developer(msg_choice, user_choice):
            cost_txts = []
            cost_txts.append(f"- การส่งข้อมูลผ่าน LINE Notify API แบบเดิม หากปรับมาใช้บริการ LINE OA โดยไม่ปรับเปลี่ยนรูปแบบข้อมูล/ผู้ใช้ คาดว่าจะมีค่าบริการรายเดือน {monthly_cost:.2f} บาท และค่าบริการข้อความเกินโควต้า {over_reach_cost:.2f} บาท รวมเป็นค่าใช้จ่ายรวม VAT เท่ากับ {lineoa_cost:.2f} บาท")
            if msg_choice['change'] == "สามารถปรับเปลี่ยนได้":
                lim_msgs = 0
                if msg_choice['emergency']:
                    cost_txts.append(f"- การจำกัดข้อความเฉพาะแจ้งเตือนเหตุการณ์ คาดว่าจะลดจำนวนเหลือไม่เกิน 1 ครั้ง/อุปกรณ์/วัน")
                    lim_msgs += 1 * num_devices * 30
                if msg_choice['status']:
                    cost_txts.append(f"- การรายงานสถานะล่าสุด คาดว่าจะเพิ่มจำนวนข้อความไม่เกิน 1 ครั้ง/อุปกรณ์/ชั่วโมง")
                    lim_msgs += 24 * num_devices * 30
                else:
                    if msg_choice['overview']:
                        cost_txts.append(f"- การจำกัดให้รายงานในระดับภาพรวม คาดว่าจะมีการรายงานไม่เกิน 2 ครั้ง/อุปกรณ์/วัน")
                        lim_msgs += 2 * num_devices * 30
            else:
                lim_msgs = num_msgs
            if user_choice['change'] == "ส่งเฉพาะผู้เกี่ยวข้อง":
                lim_users = 0
                if user_choice['core']:
                    lim_users = 1
                    cost_txts.append(f"- การจำกัดให้ส่งข้อความเฉพาะผู้รับผิดชอบหลัก คาดว่าจะมีผู้รับข้อความไม่เกิน 1 คน/อุปกรณ์")
                if user_choice['others']:
                    lim_users += (num_users-1)
                    cost_txts.append(f"- การกำหนดให้ส่งข้อความไปยังกลุ่มผู้เกี่ยวข้องอื่นๆ คาดว่าจะมีผู้รับข้อความไม่เกิน {num_users} คน/อุปกรณ์")
                else:
                    if user_choice['group']:
                        lim_users += (4 if num_users > 5 else num_users)
                        cost_txts.append(f"- การกำหนดกลุ่มผู้รับผิดชอบของแต่ละอุปกรณ์ คาดว่าจะมีผู้รับข้อความไม่เกิน {lim_users} คน/อุปกรณ์") 
            else:
                lim_users = num_users
            new_reach = lim_msgs * lim_users
            cost_txts.append(f"- การประเมินจำนวน reach จากตัวเลือกข้างบนพบว่า จำนวนผู้เกี่ยวข้อง {lim_users} คน/อุปกรณ์ และจำนวนข้อความ {lim_msgs} ครั้ง พบว่ามีจำนวน reach ใหม่เท่ากับ {new_reach} ครั้ง เทียบกับจำนวน reach เดิมคือ {total_reach}")
            return cost_txts
        
        def summarize_cost_other_apps(app_choice):
            cost_txts = []
            cost_txts.append(f"- ไม่มีค่าบริการ LINE OA")
            return cost_txts

        cost_summary = {
            ("LINE app เท่านั้น", "หน่วยงานลูกค้า"): lambda : summarize_cost_lineapp_customer(),
            ("LINE app เท่านั้น", "ผู้พัฒนา"): lambda : summarize_cost_lineapp_developer(msg_choice, user_choice),
            ("LINE app เท่านั้น", "ลูกค้า/ผู้พัฒนาจ่ายร่วม"): lambda : summarize_cost_lineapp_developer(msg_choice, user_choice),
            ("แอพอื่นบนสมาร์ทโฟน/คอมพิวเตอร์", "หน่วยงานลูกค้า"): lambda : summarize_cost_other_apps(app_choice),            
        }
        recommend_txts = cost_summary[(user_app, service_payer)]()
        st.markdown(f'''
        #### การประเมินค่าใช้จ่ายเบื้องต้น
        ''')
        for txt in recommend_txts:
            st.markdown(txt)

        st.markdown(f'''
        #### ระบบซอฟต์แวร์
        บริการ LINE messaging API มีความแตกต่างจาก LINE Notify API คือ ผู้พัฒนาต้องเตรียมระบบซอฟต์แวร์บนอินเตอร์เน็ตในรูปแบบ REST endpoint เพื่อรองรับการเชื่อมต่อจากเซิร์ฟเวอร์ของ LINE ที่จะส่งข้อมูลแบบ JSON มาทาง POST request 
        นอกจากนี้ระบบซอฟต์แวร์จะต้องเตรียมอีกหลายองค์ประกอบเพื่อทดแทนการทำงานของ LINE Notify API และองค์ประกอบที่จะส่งข้อความแจ้งเตือน/รายงานด้วยรูปแบบที่เฉพาะเจาะจง
        ''')
        def list_sw_components(msg_choice, user_choice):
            recommend_txts = []
            recommend_txts.append("1. ช่องทางรับข้อมูลและส่งคำสั่งไปยังอุปกรณ์ IoT ซึ่งอาจเลือกได้ทั้งแบบ REST endpoint (เหมือน LINE Notify API) หรือ MQTT endpoint สำหรับการสื่อสารแบบสองทาง")
            recommend_txts.append("2. ฐานข้อมูลสำหรับอุปกรณ์ IoT ซึ่งจะเก็บข้อมูลอุปกรณ์ IoT และ token สำหรับเชื่อมต่อ")
            recommend_txts.append("3. ระบบ UI สำหรับบริหารจัดการสถานะของอุปกรณ์ IoT ซึ่งควรเป็นหน้าเว็บที่ใช้งานได้ทั้งบนสมาร์ทโฟนและคอมพิวเตอร์")
            recommend_txts.append("4. ฐานข้อมูลของผู้ใช้งาน ซึ่งควรขยายไปยัง role (core, group, others) ของผู้เกี่ยวข้อง")
            recommend_txts.append("5. ระบบ UI สำหรับบริหารจัดการสิทธิของผู้ใช้งาน ซึ่งควรเป็นหน้าเว็บที่ใช้งานได้ทั้งบนสมาร์ทโฟนและคอมพิวเตอร์")
            recommend_txts.append("6. ฐานข้อมูลบันทึก flow ของข้อความเพื่อใช้ในกิจกรรม monitoring และ metering")
            recommend_txts.append("7. ช่องทางสื่อสารไปยังบริการ message แบบอื่นที่ไม่มีค่าใช้จ่าย เช่น Facebook Messenger, Discord, Telegram, Slack, MS Teams")
            recommend_txts.append("8. ระบบซอฟต์แวร์สำหรับประมวลผลข้อมูลก่อนส่งไปยัง LINE messaging API ซึ่งอาจเป็น low-code tool เช่น Node-RED")
            return recommend_txts

        for txt in list_sw_components(msg_choice, user_choice):
            st.markdown(txt)


