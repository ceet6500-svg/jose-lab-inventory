import streamlit as st
import pandas as pd
import datetime
import numpy as np
import re
import plotly.express as px
import io # For Excel Download
import matplotlib.pyplot as plt # Fix for background_gradient

# --- CAPA 容量資料 (250 筆完整數據) ---
DEFAULT_CAPA_DATA = [
    {'SHOP NUMBER': '2033', 'SUB REGION': '9F', 'SHOP NAME': 'SC PLAZA ORIENTE', 'CAPA_QTY': 429},
    {'SHOP NUMBER': '3861', 'SUB REGION': '5D', 'SHOP NAME': 'SC REVOLUCION', 'CAPA_QTY': 402},
    {'SHOP NUMBER': '4547', 'SUB REGION': '9F', 'SHOP NAME': 'SC PLAZA EDUARDO MOLINA', 'CAPA_QTY': 399},
    {'SHOP NUMBER': '2347', 'SUB REGION': '9F', 'SHOP NAME': 'SC TLAHUAC', 'CAPA_QTY': 359},
    {'SHOP NUMBER': '2344', 'SUB REGION': '9F', 'SHOP NAME': 'SC TOREO', 'CAPA_QTY': 358},
    {'SHOP NUMBER': '2345', 'SUB REGION': '9F', 'SHOP NAME': 'SC TEPEYAC', 'CAPA_QTY': 352},
    {'SHOP NUMBER': '1107', 'SUB REGION': '9F', 'SHOP NAME': 'SC IXTAPALUCA', 'CAPA_QTY': 333},
    {'SHOP NUMBER': '2079', 'SUB REGION': '9F', 'SHOP NAME': 'SC CD JARDIN', 'CAPA_QTY': 312},
    {'SHOP NUMBER': '3005', 'SUB REGION': '9F', 'SHOP NAME': 'SC PLAZA ARAGON', 'CAPA_QTY': 297},
    {'SHOP NUMBER': '3020', 'SUB REGION': '9F', 'SHOP NAME': 'BA XALOSTOC', 'CAPA_QTY': 295},
    {'SHOP NUMBER': '1489', 'SUB REGION': '9F', 'SHOP NAME': 'SC HIPERPLAZA TEXCOCO', 'CAPA_QTY': 253},
    {'SHOP NUMBER': '3799', 'SUB REGION': '9F', 'SHOP NAME': 'BA LOS ANGELES IZTAPALAPA', 'CAPA_QTY': 253},
    {'SHOP NUMBER': '3916', 'SUB REGION': '9F', 'SHOP NAME': 'BA IZTAPALAPA NORTE', 'CAPA_QTY': 249},
    {'SHOP NUMBER': '3113', 'SUB REGION': '5D', 'SHOP NAME': 'SC NILO LOMAS D L PAJAROS', 'CAPA_QTY': 248},
    {'SHOP NUMBER': '3851', 'SUB REGION': '9F', 'SHOP NAME': 'SC AEROPUERTO', 'CAPA_QTY': 247},
    {'SHOP NUMBER': '1202', 'SUB REGION': '9F', 'SHOP NAME': 'SC HORIZONTE', 'CAPA_QTY': 246},
    {'SHOP NUMBER': '1119', 'SUB REGION': '5A', 'SHOP NAME': 'SC COLON', 'CAPA_QTY': 244},
    {'SHOP NUMBER': '1423', 'SUB REGION': '9F', 'SHOP NAME': 'SC SANTA ELENA', 'CAPA_QTY': 241},
    {'SHOP NUMBER': '3753', 'SUB REGION': '9F', 'SHOP NAME': 'BA PLAZA ARAGON', 'CAPA_QTY': 240},
    {'SHOP NUMBER': '1002', 'SUB REGION': '9F', 'SHOP NAME': 'BA TEOTIHUACAN', 'CAPA_QTY': 227},
    {'SHOP NUMBER': '3884', 'SUB REGION': '9F', 'SHOP NAME': 'BA CHALCO', 'CAPA_QTY': 224},
    {'SHOP NUMBER': '3780', 'SUB REGION': '5A', 'SHOP NAME': 'BA ATEMAJAC', 'CAPA_QTY': 221},
    {'SHOP NUMBER': '3784', 'SUB REGION': '9F', 'SHOP NAME': 'BA 1 DE MAYO', 'CAPA_QTY': 218},
    {'SHOP NUMBER': '4540', 'SUB REGION': '5D', 'SHOP NAME': 'SC 16 DE SEPTIEMBRE', 'CAPA_QTY': 217},
    {'SHOP NUMBER': '4109', 'SUB REGION': '9F', 'SHOP NAME': 'SC PUERTA TEXCOCO', 'CAPA_QTY': 215},
    {'SHOP NUMBER': '1683', 'SUB REGION': '9F', 'SHOP NAME': 'SC VICENTE GUERRERO', 'CAPA_QTY': 212},
    {'SHOP NUMBER': '1171', 'SUB REGION': '9F', 'SHOP NAME': 'SC SAN JOSE TECAMAC', 'CAPA_QTY': 212},
    {'SHOP NUMBER': '3761', 'SUB REGION': '9F', 'SHOP NAME': 'BA TULYEHUALCO', 'CAPA_QTY': 207},
    {'SHOP NUMBER': '3900', 'SUB REGION': '9F', 'SHOP NAME': 'SC COACALCO', 'CAPA_QTY': 198},
    {'SHOP NUMBER': '3794', 'SUB REGION': '9F', 'SHOP NAME': 'SC ACUEDUCTO DE GPE.', 'CAPA_QTY': 198},
    {'SHOP NUMBER': '1724', 'SUB REGION': '9F', 'SHOP NAME': 'SC LOS REYES PZA ZUMPANGO', 'CAPA_QTY': 194},
    {'SHOP NUMBER': '3776', 'SUB REGION': '9F', 'SHOP NAME': 'BA SOR JUANA', 'CAPA_QTY': 190},
    {'SHOP NUMBER': '2644', 'SUB REGION': '9F', 'SHOP NAME': 'SC PATIO TLALPAN', 'CAPA_QTY': 186},
    {'SHOP NUMBER': '4138', 'SUB REGION': '4C', 'SHOP NAME': 'SC VILLA DE JUAREZ', 'CAPA_QTY': 185},
    {'SHOP NUMBER': '5813', 'SUB REGION': '9F', 'SHOP NAME': 'BA LA PAZ', 'CAPA_QTY': 183},
    {'SHOP NUMBER': '2381', 'SUB REGION': '5B', 'SHOP NAME': 'SC MORELIA', 'CAPA_QTY': 182},
    {'SHOP NUMBER': '3857', 'SUB REGION': '9F', 'SHOP NAME': 'SC VILLA COAPA', 'CAPA_QTY': 182},
    {'SHOP NUMBER': '3874', 'SUB REGION': '9F', 'SHOP NAME': 'BA CANTIL', 'CAPA_QTY': 180},
    {'SHOP NUMBER': '2670', 'SUB REGION': '9F', 'SHOP NAME': 'SC TLALPAN', 'CAPA_QTY': 179},
    {'SHOP NUMBER': '5733', 'SUB REGION': '9F', 'SHOP NAME': 'BA PLAZA CHIMALHUACAN', 'CAPA_QTY': 179},
    {'SHOP NUMBER': '1432', 'SUB REGION': '8C', 'SHOP NAME': 'MB OCOSINGO', 'CAPA_QTY': 177},
    {'SHOP NUMBER': '3785', 'SUB REGION': '9C', 'SHOP NAME': 'BA CUAUTLA', 'CAPA_QTY': 173},
    {'SHOP NUMBER': '1177', 'SUB REGION': '9F', 'SHOP NAME': 'BA S TIANGUISTENCO', 'CAPA_QTY': 172},
    {'SHOP NUMBER': '3775', 'SUB REGION': '9F', 'SHOP NAME': 'BA LA AURORA', 'CAPA_QTY': 172},
    {'SHOP NUMBER': '3846', 'SUB REGION': '9F', 'SHOP NAME': 'SC BUENAVISTA', 'CAPA_QTY': 168},
    {'SHOP NUMBER': '2643', 'SUB REGION': '9F', 'SHOP NAME': 'SC NICOLAS ROMERO', 'CAPA_QTY': 168},
    {'SHOP NUMBER': '3503', 'SUB REGION': '9F', 'SHOP NAME': 'SC ECATEPEC CENTRO', 'CAPA_QTY': 168},
    {'SHOP NUMBER': '3797', 'SUB REGION': '9F', 'SHOP NAME': 'BA CABEZA DE JUAREZ', 'CAPA_QTY': 168},
    {'SHOP NUMBER': '2994', 'SUB REGION': '6B', 'SHOP NAME': 'BA DUARTE', 'CAPA_QTY': 168},
    {'SHOP NUMBER': '2464', 'SUB REGION': '9F', 'SHOP NAME': 'SC MIRAMONTES', 'CAPA_QTY': 167},
    {'SHOP NUMBER': '3771', 'SUB REGION': '9F', 'SHOP NAME': 'BA PLAZA CHURUBUSCO', 'CAPA_QTY': 166},
    {'SHOP NUMBER': '1624', 'SUB REGION': '8C', 'SHOP NAME': 'SC COMITAN', 'CAPA_QTY': 165},
    {'SHOP NUMBER': '4090', 'SUB REGION': '9C', 'SHOP NAME': 'SC LOS ATRIOS', 'CAPA_QTY': 165},
    {'SHOP NUMBER': '1045', 'SUB REGION': '9C', 'SHOP NAME': 'SC DOMINGO DIEZ', 'CAPA_QTY': 165},
    {'SHOP NUMBER': '1086', 'SUB REGION': '6B', 'SHOP NAME': 'BA DELTA', 'CAPA_QTY': 162},
    {'SHOP NUMBER': '1132', 'SUB REGION': '5B', 'SHOP NAME': 'BA PATZCUARO', 'CAPA_QTY': 160},
    {'SHOP NUMBER': '3862', 'SUB REGION': '9F', 'SHOP NAME': 'SC AZCAPOTZALCO', 'CAPA_QTY': 159},
    {'SHOP NUMBER': '3877', 'SUB REGION': '9F', 'SHOP NAME': 'SC TORRES LINDAVISTA', 'CAPA_QTY': 155},
    {'SHOP NUMBER': '3038', 'SUB REGION': '4A', 'SHOP NAME': 'BA CD MANTE', 'CAPA_QTY': 155},
    {'SHOP NUMBER': '2648', 'SUB REGION': '7A', 'SHOP NAME': 'SC PLAZA EL PASEO', 'CAPA_QTY': 155},
    {'SHOP NUMBER': '3744', 'SUB REGION': '6B', 'SHOP NAME': 'BA OJO CALIENTE', 'CAPA_QTY': 154},
    {'SHOP NUMBER': '2734', 'SUB REGION': '5B', 'SHOP NAME': 'SC PLAZA ANA', 'CAPA_QTY': 154},
    {'SHOP NUMBER': '3764', 'SUB REGION': '9F', 'SHOP NAME': 'BA IZTAPALAPA', 'CAPA_QTY': 152},
    {'SHOP NUMBER': '2246', 'SUB REGION': '8C', 'SHOP NAME': 'MB LACANDONA', 'CAPA_QTY': 151},
    {'SHOP NUMBER': '2736', 'SUB REGION': '9F', 'SHOP NAME': 'BA CHIMALHUACAN PENON', 'CAPA_QTY': 151},
    {'SHOP NUMBER': '3631', 'SUB REGION': '5D', 'SHOP NAME': 'SC TONALA', 'CAPA_QTY': 150},
    {'SHOP NUMBER': '2346', 'SUB REGION': '6B', 'SHOP NAME': 'SC AGUASCALIENTES', 'CAPA_QTY': 150},
    {'SHOP NUMBER': '2054', 'SUB REGION': '9F', 'SHOP NAME': 'BA TERRANOVA', 'CAPA_QTY': 148},
    {'SHOP NUMBER': '3355', 'SUB REGION': '5B', 'SHOP NAME': 'SC ESTADIO', 'CAPA_QTY': 148},
    {'SHOP NUMBER': '1109', 'SUB REGION': '9F', 'SHOP NAME': 'BA AYOTLA', 'CAPA_QTY': 146},
    {'SHOP NUMBER': '3848', 'SUB REGION': '9F', 'SHOP NAME': 'SC TAXQUENA', 'CAPA_QTY': 146},
    {'SHOP NUMBER': '5727', 'SUB REGION': '9C', 'SHOP NAME': 'SC JIUTEPEC', 'CAPA_QTY': 145},
    {'SHOP NUMBER': '2525', 'SUB REGION': '9F', 'SHOP NAME': 'BA RIO HONDO', 'CAPA_QTY': 143},
    {'SHOP NUMBER': '3777', 'SUB REGION': '9F', 'SHOP NAME': 'BA VALLE DE ARAGON', 'CAPA_QTY': 143},
    {'SHOP NUMBER': '3845', 'SUB REGION': '9F', 'SHOP NAME': 'SC UNIVERSIDAD', 'CAPA_QTY': 143},
    {'SHOP NUMBER': '3751', 'SUB REGION': '9F', 'SHOP NAME': 'BA INSURGENTES SUR', 'CAPA_QTY': 142},
    {'SHOP NUMBER': '5825', 'SUB REGION': '9F', 'SHOP NAME': 'SC ALFREDO DEL MAZO', 'CAPA_QTY': 142},
    {'SHOP NUMBER': '3042', 'SUB REGION': '5A', 'SHOP NAME': 'BA SANTA PAULA', 'CAPA_QTY': 141},
    {'SHOP NUMBER': '1032', 'SUB REGION': '9F', 'SHOP NAME': 'SC LOMAS VERDES', 'CAPA_QTY': 140},
    {'SHOP NUMBER': '3791', 'SUB REGION': '9F', 'SHOP NAME': 'BA FTES. DEL VALLE', 'CAPA_QTY': 140},
    {'SHOP NUMBER': '2074', 'SUB REGION': '6C', 'SHOP NAME': 'SC CARRETERA 57', 'CAPA_QTY': 140},
    {'SHOP NUMBER': '2075', 'SUB REGION': '6C', 'SHOP NAME': 'SC SLP MUNOZ', 'CAPA_QTY': 139},
    {'SHOP NUMBER': '3850', 'SUB REGION': '9F', 'SHOP NAME': 'SC ECHEGARAY', 'CAPA_QTY': 138},
    {'SHOP NUMBER': '3795', 'SUB REGION': '9F', 'SHOP NAME': 'BA TEXCOCO', 'CAPA_QTY': 137},
    {'SHOP NUMBER': '3239', 'SUB REGION': '4C', 'SHOP NAME': 'SC PASEO REAL', 'CAPA_QTY': 137},
    {'SHOP NUMBER': '3770', 'SUB REGION': '9F', 'SHOP NAME': 'BA PANTITLAN', 'CAPA_QTY': 137},
    {'SHOP NUMBER': '3858', 'SUB REGION': '9F', 'SHOP NAME': 'SC CUAJIMALPA', 'CAPA_QTY': 136},
    {'SHOP NUMBER': '2141', 'SUB REGION': '9F', 'SHOP NAME': 'BA XONACATLAN', 'CAPA_QTY': 136},
    {'SHOP NUMBER': '3872', 'SUB REGION': '9F', 'SHOP NAME': 'SC BALBUENA', 'CAPA_QTY': 135},
    {'SHOP NUMBER': '2737', 'SUB REGION': '9F', 'SHOP NAME': 'BA ETZATLAN', 'CAPA_QTY': 135},
    {'SHOP NUMBER': '1580', 'SUB REGION': '9F', 'SHOP NAME': 'SC EL ROSARIO', 'CAPA_QTY': 135},
    {'SHOP NUMBER': '3790', 'SUB REGION': '9F', 'SHOP NAME': 'SC METEPEC', 'CAPA_QTY': 134},
    {'SHOP NUMBER': '3768', 'SUB REGION': '9F', 'SHOP NAME': 'BA VILLA COAPA', 'CAPA_QTY': 133},
    {'SHOP NUMBER': '3778', 'SUB REGION': '9F', 'SHOP NAME': 'BA XOCHIMILCO', 'CAPA_QTY': 132},
    {'SHOP NUMBER': '4191', 'SUB REGION': '9F', 'SHOP NAME': 'SC LAGO DE GUADALUPE', 'CAPA_QTY': 132},
    {'SHOP NUMBER': '2343', 'SUB REGION': '9F', 'SHOP NAME': 'SC TOLUCA', 'CAPA_QTY': 132},
    {'SHOP NUMBER': '1044', 'SUB REGION': '9F', 'SHOP NAME': 'SC SAN MARCOS IZCALLI', 'CAPA_QTY': 132},
    {'SHOP NUMBER': '3722', 'SUB REGION': '9C', 'SHOP NAME': 'BA TULANCINGO', 'CAPA_QTY': 130},
    {'SHOP NUMBER': '3755', 'SUB REGION': '9F', 'SHOP NAME': 'BA TACUBAYA', 'CAPA_QTY': 129},
    {'SHOP NUMBER': '1104', 'SUB REGION': '9F', 'SHOP NAME': 'BA TIZAYUCA', 'CAPA_QTY': 128},
    {'SHOP NUMBER': '2466', 'SUB REGION': '9F', 'SHOP NAME': 'SC CUITLAHUAC', 'CAPA_QTY': 127},
    {'SHOP NUMBER': '1083', 'SUB REGION': '9F', 'SHOP NAME': 'SC PORTAL SAN ANGEL', 'CAPA_QTY': 126},
    {'SHOP NUMBER': '2947', 'SUB REGION': 'R1', 'SHOP NAME': 'SC ENSENADA CENTRO', 'CAPA_QTY': 126},
    {'SHOP NUMBER': '460', 'SUB REGION': '6B', 'SHOP NAME': 'BA PASEO LAS TORRES', 'CAPA_QTY': 126},
    {'SHOP NUMBER': '1073', 'SUB REGION': '5A', 'SHOP NAME': 'BA OCOTLAN JALISCO', 'CAPA_QTY': 125},
    {'SHOP NUMBER': '3016', 'SUB REGION': '9F', 'SHOP NAME': 'SC MACRO PLAZA HEROES', 'CAPA_QTY': 125},
    {'SHOP NUMBER': '2080', 'SUB REGION': '6C', 'SHOP NAME': 'SC SLP ARBOLEDAS', 'CAPA_QTY': 125},
    {'SHOP NUMBER': '1634', 'SUB REGION': '9F', 'SHOP NAME': 'BA CUAUTZINGO CHALCO', 'CAPA_QTY': 125},
    {'SHOP NUMBER': '1018', 'SUB REGION': '9C', 'SHOP NAME': 'BA TEMIXCO II', 'CAPA_QTY': 125},
    {'SHOP NUMBER': '4558', 'SUB REGION': '9F', 'SHOP NAME': 'BA TECAMAC', 'CAPA_QTY': 124},
    {'SHOP NUMBER': '3805', 'SUB REGION': '9F', 'SHOP NAME': 'BA SANTA LUCIA', 'CAPA_QTY': 124},
    {'SHOP NUMBER': '4628', 'SUB REGION': '9F', 'SHOP NAME': 'SC LAS ANTENAS', 'CAPA_QTY': 124},
    {'SHOP NUMBER': '3061', 'SUB REGION': '9C', 'SHOP NAME': 'SC PACHUCA', 'CAPA_QTY': 124},
    {'SHOP NUMBER': '5765', 'SUB REGION': '9F', 'SHOP NAME': 'SC LAS AMERICAS', 'CAPA_QTY': 123},
    {'SHOP NUMBER': '1902', 'SUB REGION': '4A', 'SHOP NAME': 'SC AEROPUERTO TAMPICO', 'CAPA_QTY': 123},
    {'SHOP NUMBER': '1150', 'SUB REGION': '9F', 'SHOP NAME': 'BA HUEHUETOCA', 'CAPA_QTY': 122},
    {'SHOP NUMBER': '4155', 'SUB REGION': 'R1', 'SHOP NAME': 'SC TIJUANA 2000', 'CAPA_QTY': 122},
    {'SHOP NUMBER': '3863', 'SUB REGION': '9F', 'SHOP NAME': 'SC PERIFERICO SUR', 'CAPA_QTY': 122},
    {'SHOP NUMBER': '1828', 'SUB REGION': '9F', 'SHOP NAME': 'BA ACOLMAN TEPEXPAN', 'CAPA_QTY': 121},
    {'SHOP NUMBER': '1181', 'SUB REGION': '6B', 'SHOP NAME': 'BA LEON ECHEVESTE', 'CAPA_QTY': 121},
    {'SHOP NUMBER': '3257', 'SUB REGION': '5D', 'SHOP NAME': 'SC TEPATITLAN', 'CAPA_QTY': 119},
    {'SHOP NUMBER': '5770', 'SUB REGION': '9F', 'SHOP NAME': 'BA ZUMPANGO', 'CAPA_QTY': 118},
    {'SHOP NUMBER': '4072', 'SUB REGION': '8C', 'SHOP NAME': 'SC TAPACHULA CHIAPAS', 'CAPA_QTY': 118},
    {'SHOP NUMBER': '1022', 'SUB REGION': '5A', 'SHOP NAME': 'SC LOPEZ MATEOS SUR', 'CAPA_QTY': 118},
    {'SHOP NUMBER': '2342', 'SUB REGION': '5A', 'SHOP NAME': 'SC VALLARTA', 'CAPA_QTY': 117},
    {'SHOP NUMBER': '1201', 'SUB REGION': '9F', 'SHOP NAME': 'BA SAN AGUSTIN', 'CAPA_QTY': 115},
    {'SHOP NUMBER': '1007', 'SUB REGION': '4B', 'SHOP NAME': 'SC HAROLD PAPE MONCLOVA', 'CAPA_QTY': 115},
    {'SHOP NUMBER': '3783', 'SUB REGION': '9F', 'SHOP NAME': 'BA SANTA CLARA', 'CAPA_QTY': 114},
    {'SHOP NUMBER': '3767', 'SUB REGION': '9C', 'SHOP NAME': 'BA PACHUCA', 'CAPA_QTY': 114},
    {'SHOP NUMBER': '5798', 'SUB REGION': '5A', 'SHOP NAME': 'BA SANTA MARGARITA', 'CAPA_QTY': 114},
    {'SHOP NUMBER': '2732', 'SUB REGION': '7A', 'SHOP NAME': 'SC HOSPITAL GENERAL', 'CAPA_QTY': 114},
    {'SHOP NUMBER': '5040', 'SUB REGION': '9F', 'SHOP NAME': 'SC JIMENEZ CANTU SPCT', 'CAPA_QTY': 113},
    {'SHOP NUMBER': '3033', 'SUB REGION': '3A', 'SHOP NAME': 'SC FUENTES MARES', 'CAPA_QTY': 112},
    {'SHOP NUMBER': '1161', 'SUB REGION': '9F', 'SHOP NAME': 'BA VALLE DE CHALCO', 'CAPA_QTY': 112},
    {'SHOP NUMBER': '3781', 'SUB REGION': '5D', 'SHOP NAME': 'BA INDEPENDENCIA', 'CAPA_QTY': 111},
    {'SHOP NUMBER': '3852', 'SUB REGION': '9F', 'SHOP NAME': 'SC PLATEROS', 'CAPA_QTY': 110},
    {'SHOP NUMBER': '3897', 'SUB REGION': '9F', 'SHOP NAME': 'BA LA VIRGEN', 'CAPA_QTY': 110},
    {'SHOP NUMBER': '3860', 'SUB REGION': '9F', 'SHOP NAME': 'BA ZARAGOZA', 'CAPA_QTY': 109},
    {'SHOP NUMBER': '3664', 'SUB REGION': 'R1', 'SHOP NAME': 'SC DIAZ ORDAZ', 'CAPA_QTY': 108},
    {'SHOP NUMBER': '1728', 'SUB REGION': '8C', 'SHOP NAME': 'BA SARAGUATO', 'CAPA_QTY': 108},
    {'SHOP NUMBER': '3905', 'SUB REGION': '6B', 'SHOP NAME': 'BA SANTA ANITA', 'CAPA_QTY': 108},
    {'SHOP NUMBER': '3774', 'SUB REGION': '9F', 'SHOP NAME': 'BA F.F.C.C. HIDALGO', 'CAPA_QTY': 107},
    {'SHOP NUMBER': '4542', 'SUB REGION': '5B', 'SHOP NAME': 'BA MORELIA TRES PUENTES', 'CAPA_QTY': 107},
    {'SHOP NUMBER': '2666', 'SUB REGION': '9C', 'SHOP NAME': 'BA JOJUTLA', 'CAPA_QTY': 106},
    {'SHOP NUMBER': '3756', 'SUB REGION': '9F', 'SHOP NAME': 'BA TLALNEPANTLA', 'CAPA_QTY': 105},
    {'SHOP NUMBER': '2349', 'SUB REGION': '7B', 'SHOP NAME': 'SC ACAPULCO', 'CAPA_QTY': 105},
    {'SHOP NUMBER': '3893', 'SUB REGION': '7C', 'SHOP NAME': 'SC XALAPA', 'CAPA_QTY': 105},
    {'SHOP NUMBER': '3523', 'SUB REGION': '8A', 'SHOP NAME': 'BA NUEVA KUKULCAN', 'CAPA_QTY': 105},
    {'SHOP NUMBER': '2433', 'SUB REGION': '6B', 'SHOP NAME': 'SC LEON TORRES LANDA', 'CAPA_QTY': 105},
    {'SHOP NUMBER': '3798', 'SUB REGION': '9F', 'SHOP NAME': 'BA SANTA CECILIA', 'CAPA_QTY': 104},
    {'SHOP NUMBER': '3803', 'SUB REGION': '9F', 'SHOP NAME': 'BA OBSERVATORIO', 'CAPA_QTY': 103},
    {'SHOP NUMBER': '4018', 'SUB REGION': '9F', 'SHOP NAME': 'SC SANTA MARIA', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '3922', 'SUB REGION': '6A', 'SHOP NAME': 'BA EL TINTERO', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '3127', 'SUB REGION': '8C', 'SHOP NAME': 'SC LIBRAMIENTO NORTE', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '3766', 'SUB REGION': '9F', 'SHOP NAME': 'BA LA VIGA', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '5469', 'SUB REGION': '9F', 'SHOP NAME': 'SC RIO DE LOS REMEDIOS', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '1400', 'SUB REGION': '9F', 'SHOP NAME': 'BA JOYAS DE COACALCO', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '3806', 'SUB REGION': '7B', 'SHOP NAME': 'BA RENACIMIENTO ACAPULCO', 'CAPA_QTY': 102},
    {'SHOP NUMBER': '3719', 'SUB REGION': '4A', 'SHOP NAME': 'SC MATAMOROS', 'CAPA_QTY': 101},
    {'SHOP NUMBER': '3721', 'SUB REGION': '5D', 'SHOP NAME': 'SC AVILA CAMACHO', 'CAPA_QTY': 98},
    {'SHOP NUMBER': '1019', 'SUB REGION': '5A', 'SHOP NAME': 'BA SAN PEDRITO', 'CAPA_QTY': 98},
    {'SHOP NUMBER': '1663', 'SUB REGION': '9C', 'SHOP NAME': 'BA EL MANANTIAL', 'CAPA_QTY': 98},
    {'SHOP NUMBER': '2731', 'SUB REGION': '7A', 'SHOP NAME': 'SC EL MOLINITO TLAXCALA', 'CAPA_QTY': 97},
    {'SHOP NUMBER': '1649', 'SUB REGION': '8C', 'SHOP NAME': 'MB CACAHOATAN', 'CAPA_QTY': 97},
    {'SHOP NUMBER': '5791', 'SUB REGION': '9F', 'SHOP NAME': 'SC ZINACANTEPEC', 'CAPA_QTY': 96},
    {'SHOP NUMBER': '2768', 'SUB REGION': '7A', 'SHOP NAME': 'BA AGUSTIN LARA', 'CAPA_QTY': 95},
    {'SHOP NUMBER': '4541', 'SUB REGION': '9F', 'SHOP NAME': 'BA IXTAPALUCA', 'CAPA_QTY': 95},
    {'SHOP NUMBER': '2136', 'SUB REGION': '6B', 'SHOP NAME': 'BA BOULEVARD MORELOS', 'CAPA_QTY': 95},
    {'SHOP NUMBER': '3588', 'SUB REGION': '9C', 'SHOP NAME': 'SC PARAISO', 'CAPA_QTY': 95},
    {'SHOP NUMBER': '2072', 'SUB REGION': '9F', 'SHOP NAME': 'BA PORTAL CHALCO', 'CAPA_QTY': 95},
    {'SHOP NUMBER': '3031', 'SUB REGION': '6B', 'SHOP NAME': 'SC CELAYA IRRIGACION', 'CAPA_QTY': 94},
    {'SHOP NUMBER': '2223', 'SUB REGION': '4A', 'SHOP NAME': 'BA REYNOSA', 'CAPA_QTY': 94},
    {'SHOP NUMBER': '2219', 'SUB REGION': '9F', 'SHOP NAME': 'SC HUEHUETOCA JOROBAS', 'CAPA_QTY': 93},
    {'SHOP NUMBER': '5657', 'SUB REGION': '6C', 'SHOP NAME': 'BA SOLEDAD DE GRACIANO', 'CAPA_QTY': 93},
    {'SHOP NUMBER': '1194', 'SUB REGION': '9F', 'SHOP NAME': 'MB IXTLAHUACA', 'CAPA_QTY': 93},
    {'SHOP NUMBER': '1142', 'SUB REGION': '9C', 'SHOP NAME': 'BA TOTOLTEPEC', 'CAPA_QTY': 92},
    {'SHOP NUMBER': '2387', 'SUB REGION': '9C', 'SHOP NAME': 'BA HACIENDA DE TIZAYUCA', 'CAPA_QTY': 92},
    {'SHOP NUMBER': '1176', 'SUB REGION': '5D', 'SHOP NAME': 'BA SN JUAN D LOS LAGOS', 'CAPA_QTY': 91},
    {'SHOP NUMBER': '3782', 'SUB REGION': '9F', 'SHOP NAME': 'BA CENTENARIO', 'CAPA_QTY': 91},
    {'SHOP NUMBER': '3883', 'SUB REGION': '9F', 'SHOP NAME': 'BA MELCHOR OCAMPO', 'CAPA_QTY': 89},
    {'SHOP NUMBER': '2041', 'SUB REGION': '9F', 'SHOP NAME': 'SC TOLTECAS', 'CAPA_QTY': 89},
    {'SHOP NUMBER': '3043', 'SUB REGION': '9F', 'SHOP NAME': 'BA ZINACANTEPEC', 'CAPA_QTY': 89},
    {'SHOP NUMBER': '5687', 'SUB REGION': '6B', 'SHOP NAME': 'SC BOULEVARD DELTA', 'CAPA_QTY': 89},
    {'SHOP NUMBER': '1870', 'SUB REGION': '9F', 'SHOP NAME': 'BA TEPOTZOTLAN', 'CAPA_QTY': 88},
    {'SHOP NUMBER': '1684', 'SUB REGION': '7B', 'SHOP NAME': 'SC CHILPANCINGO', 'CAPA_QTY': 88},
    {'SHOP NUMBER': '3773', 'SUB REGION': '7A', 'SHOP NAME': 'BA VIA CAPU', 'CAPA_QTY': 88},
    {'SHOP NUMBER': '2424', 'SUB REGION': '6B', 'SHOP NAME': 'BA ARISTOTELES', 'CAPA_QTY': 88},
    {'SHOP NUMBER': '1519', 'SUB REGION': '9F', 'SHOP NAME': 'MB TLALMANALCO', 'CAPA_QTY': 88},
    {'SHOP NUMBER': '3876', 'SUB REGION': '9F', 'SHOP NAME': 'SC LAS AGUILAS', 'CAPA_QTY': 87},
    {'SHOP NUMBER': '3015', 'SUB REGION': 'R1', 'SHOP NAME': 'SC ENSENADA', 'CAPA_QTY': 87},
    {'SHOP NUMBER': '1236', 'SUB REGION': '9F', 'SHOP NAME': 'BD ATLACOMULCO', 'CAPA_QTY': 87},
    {'SHOP NUMBER': '1067', 'SUB REGION': '7B', 'SHOP NAME': 'SC DIAMANTE', 'CAPA_QTY': 87},
    {'SHOP NUMBER': '5855', 'SUB REGION': '9F', 'SHOP NAME': 'SC LAS ALAMEDAS', 'CAPA_QTY': 86},
    {'SHOP NUMBER': '2999', 'SUB REGION': '5A', 'SHOP NAME': 'BA TALA RUISENORES', 'CAPA_QTY': 85},
    {'SHOP NUMBER': '3763', 'SUB REGION': '9F', 'SHOP NAME': 'BA MARIANO ESCOBEDO', 'CAPA_QTY': 85},
    {'SHOP NUMBER': '3294', 'SUB REGION': '9F', 'SHOP NAME': 'BA NICOLAS BRAVO', 'CAPA_QTY': 84},
    {'SHOP NUMBER': '1462', 'SUB REGION': '9F', 'SHOP NAME': 'SC LERMA TOLUCA', 'CAPA_QTY': 84},
    {'SHOP NUMBER': '3004', 'SUB REGION': '7A', 'SHOP NAME': 'BA HUAUCHINANGO', 'CAPA_QTY': 83},
    {'SHOP NUMBER': '5713', 'SUB REGION': '7A', 'SHOP NAME': 'BA TLAXCALA', 'CAPA_QTY': 83},
    {'SHOP NUMBER': '1205', 'SUB REGION': '4A', 'SHOP NAME': 'SC ALIJADORES', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '1001', 'SUB REGION': '9F', 'SHOP NAME': 'BA SAN BUENAVENTURA', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '3622', 'SUB REGION': '4B', 'SHOP NAME': 'SC LA ROSITA', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '5712', 'SUB REGION': '4B', 'SHOP NAME': 'BA SOLIDARIDAD', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '3074', 'SUB REGION': '9F', 'SHOP NAME': 'BA VILLAS DE LA HACIENDA', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '1501', 'SUB REGION': '9F', 'SHOP NAME': 'MB OZUMBA', 'CAPA_QTY': 82},
    {'SHOP NUMBER': '3891', 'SUB REGION': '9F', 'SHOP NAME': 'BA LOS REYES', 'CAPA_QTY': 81},
    {'SHOP NUMBER': '2430', 'SUB REGION': '9F', 'SHOP NAME': 'SC COPILCO', 'CAPA_QTY': 81},
    {'SHOP NUMBER': '3909', 'SUB REGION': '7C', 'SHOP NAME': 'SC CORDOBA', 'CAPA_QTY': 80},
    {'SHOP NUMBER': '3143', 'SUB REGION': '5A', 'SHOP NAME': 'BA LOS AGAVES', 'CAPA_QTY': 80},
    {'SHOP NUMBER': '4081', 'SUB REGION': '9C', 'SHOP NAME': 'MB TLAXCOAPAN', 'CAPA_QTY': 80},
    {'SHOP NUMBER': '2380', 'SUB REGION': '2B', 'SHOP NAME': 'SC CD. OBREGON', 'CAPA_QTY': 79},
    {'SHOP NUMBER': '3902', 'SUB REGION': '5B', 'SHOP NAME': 'BA LAZARO CARDENAS', 'CAPA_QTY': 79},
    {'SHOP NUMBER': '1830', 'SUB REGION': '9F', 'SHOP NAME': 'BA ARBOLADA LS SAUCES', 'CAPA_QTY': 78},
    {'SHOP NUMBER': '4017', 'SUB REGION': '9F', 'SHOP NAME': 'BA TOLUCA AZTECAS', 'CAPA_QTY': 78},
    {'SHOP NUMBER': '3585', 'SUB REGION': '8C', 'SHOP NAME': 'MB SAN FERNANDO CHIAPAS', 'CAPA_QTY': 78},
    {'SHOP NUMBER': '5605', 'SUB REGION': '8C', 'SHOP NAME': 'MB ESTACION', 'CAPA_QTY': 77},
    {'SHOP NUMBER': '1404', 'SUB REGION': '4B', 'SHOP NAME': 'SC GALERIAS SALTILLO', 'CAPA_QTY': 77},
    {'SHOP NUMBER': '4168', 'SUB REGION': '6B', 'SHOP NAME': 'MB JUVENTINO ROSAS', 'CAPA_QTY': 76},
    {'SHOP NUMBER': '3907', 'SUB REGION': '7B', 'SHOP NAME': 'BA COLOSO II', 'CAPA_QTY': 76},
    {'SHOP NUMBER': '1829', 'SUB REGION': '9F', 'SHOP NAME': 'BA ALMOLOYA', 'CAPA_QTY': 76},
    {'SHOP NUMBER': '3668', 'SUB REGION': '9F', 'SHOP NAME': 'BA REAL DE COSTITLAN', 'CAPA_QTY': 75},
    {'SHOP NUMBER': '3363', 'SUB REGION': '9C', 'SHOP NAME': 'MB TLALCILALCALPAN', 'CAPA_QTY': 75},
    {'SHOP NUMBER': '3167', 'SUB REGION': '9F', 'SHOP NAME': 'BA SANTA INES', 'CAPA_QTY': 74},
    {'SHOP NUMBER': '3887', 'SUB REGION': '6C', 'SHOP NAME': 'BA RIO VERDE SLP', 'CAPA_QTY': 74},
    {'SHOP NUMBER': '5662', 'SUB REGION': '9C', 'SHOP NAME': 'BA TENANCINGO', 'CAPA_QTY': 74},
    {'SHOP NUMBER': '1035', 'SUB REGION': '7A', 'SHOP NAME': 'BA FORJADORES', 'CAPA_QTY': 73},
    {'SHOP NUMBER': '1111', 'SUB REGION': '9F', 'SHOP NAME': 'BA EL ALAMO', 'CAPA_QTY': 73},
    {'SHOP NUMBER': '5089', 'SUB REGION': '7A', 'SHOP NAME': 'SC SAN MARTIN TEXMELUCAN', 'CAPA_QTY': 72},
    {'SHOP NUMBER': '2783', 'SUB REGION': '9C', 'SHOP NAME': 'MB VILLA GUERRERO', 'CAPA_QTY': 72},
    {'SHOP NUMBER': '3364', 'SUB REGION': '9C', 'SHOP NAME': 'MB TEJUPILCO CRISTOBAL', 'CAPA_QTY': 72},
    {'SHOP NUMBER': '5749', 'SUB REGION': '6A', 'SHOP NAME': 'SC PLAZA DE TOROS', 'CAPA_QTY': 71},
    {'SHOP NUMBER': '3890', 'SUB REGION': '5B', 'SHOP NAME': 'BA LA PIEDAD', 'CAPA_QTY': 71},
    {'SHOP NUMBER': '3068', 'SUB REGION': '6C', 'SHOP NAME': 'BA CIUDAD RIO VERDE', 'CAPA_QTY': 71},
    {'SHOP NUMBER': '1412', 'SUB REGION': '6B', 'SHOP NAME': 'MB VILLAGRAN', 'CAPA_QTY': 70},
    {'SHOP NUMBER': '3487', 'SUB REGION': '9C', 'SHOP NAME': 'MB TEQUIXQUIAC', 'CAPA_QTY': 69},
    {'SHOP NUMBER': '2240', 'SUB REGION': '9C', 'SHOP NAME': 'BA STA CRUZ', 'CAPA_QTY': 68},
    {'SHOP NUMBER': '1345', 'SUB REGION': '9F', 'SHOP NAME': 'BA LAS BRISAS CUAUTLA', 'CAPA_QTY': 68},
    {'SHOP NUMBER': '3769', 'SUB REGION': '9F', 'SHOP NAME': 'BA INSURGENTES - NORTE', 'CAPA_QTY': 68},
    {'SHOP NUMBER': '1428', 'SUB REGION': '9F', 'SHOP NAME': 'BA SAN MATEO ATENCO', 'CAPA_QTY': 67},
    {'SHOP NUMBER': '3750', 'SUB REGION': '9F', 'SHOP NAME': 'BA AUTOPISTA QRO.', 'CAPA_QTY': 67},
    {'SHOP NUMBER': '3400', 'SUB REGION': 'R1', 'SHOP NAME': 'BA DELICIAS', 'CAPA_QTY': 66},
    {'SHOP NUMBER': '1899', 'SUB REGION': '9C', 'SHOP NAME': 'BA SAN JUAN DE LA LABOR', 'CAPA_QTY': 66},
    {'SHOP NUMBER': '2544', 'SUB REGION': '3B', 'SHOP NAME': 'BD PRIMO DE VERDAD', 'CAPA_QTY': 66},
    {'SHOP NUMBER': '5182', 'SUB REGION': '9F', 'SHOP NAME': 'BD BICENTENARIO', 'CAPA_QTY': 66},
    {'SHOP NUMBER': '3762', 'SUB REGION': '9F', 'SHOP NAME': 'BA CHIMALHUACAN', 'CAPA_QTY': 66},
    {'SHOP NUMBER': '5730', 'SUB REGION': '9F', 'SHOP NAME': 'BA CHICOLOAPAN', 'CAPA_QTY': 65},
    {'SHOP NUMBER': '1693', 'SUB REGION': '8B', 'SHOP NAME': 'BA JALPA DE MENDEZ', 'CAPA_QTY': 65},
    {'SHOP NUMBER': '1961', 'SUB REGION': '9F', 'SHOP NAME': 'MB VILLA VICTORIA', 'CAPA_QTY': 65},
    {'SHOP NUMBER': '4825', 'SUB REGION': '5A', 'SHOP NAME': 'BD TESISTAN CENTRO', 'CAPA_QTY': 65},
    {'SHOP NUMBER': '2281', 'SUB REGION': '9C', 'SHOP NAME': 'MB JESUS CHAPARRO', 'CAPA_QTY': 64}
]

# --- 1. Configuración de Página y Título ---
st.set_page_config(
    page_title="JOSE LAB.", 
    layout="wide",
    initial_sidebar_state="expanded"
)

# --- 2. CSS: SEE+SAW Gallery Style (Strict) & Ticker Animation ---
st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;900&display=swap');

    .stApp {
        background-color: #FFFFFF;
        color: #000000;
        font-family: 'Inter', sans-serif;
    }
    
    [data-testid="stSidebar"] {
        background-color: #F7F7F7;
        border-right: 1px solid #E5E5E5;
    }

    h1, h2, h3 {
        color: #000000 !important;
        font-weight: 900 !important;
        letter-spacing: -0.05rem;
    }
    
    /* Ticker CSS */
    .ticker-wrap {
        width: 100%;
        overflow: hidden;
        background-color: #000000;
        padding-left: 100%;
        box-sizing: content-box;
        margin-bottom: 20px;
    }
    .ticker {
        display: inline-block;
        white-space: nowrap;
        padding-right: 100%;
        box-sizing: content-box;
        animation-iteration-count: infinite;
        animation-timing-function: linear;
        animation-name: ticker;
        animation-duration: 45s;
    }
    .ticker__item {
        display: inline-block;
        padding: 0 2rem;
        font-size: 1rem;
        color: #FFFFFF;
        font-weight: 600;
    }
    @keyframes ticker {
        0% { transform: translate3d(0, 0, 0); }
        100% { transform: translate3d(-100%, 0, 0); }
    }

    [data-testid="stMetricValue"] {
        color: #000000 !important;
        font-size: 2.0rem !important; 
        font-weight: 900 !important;
        line-height: 1.1;
        overflow-wrap: break-word;
    }
    
    thead tr th {
        background-color: #000000 !important;
        color: #FFFFFF !important;
        font-weight: 700 !important;
        text-transform: uppercase;
        border: none;
    }
    tbody tr td {
        color: #000000 !important;
        border-bottom: 1px solid #E5E5E5 !important;
        font-weight: 500;
    }

    .stTabs [data-baseweb="tab-list"] {
        gap: 2rem;
    }
    .stTabs [data-baseweb="tab"] {
        font-weight: 600;
        color: #999999;
        border: none;
    }
    .stTabs [data-baseweb="tab"][aria-selected="true"] {
        color: #000000;
        font-weight: 900;
        border-bottom: 3px solid #000000;
        background-color: transparent;
    }
    
    hr {
        border-top: 2px solid #000000;
        border-bottom: none;
        opacity: 1;
    }
</style>
""", unsafe_allow_html=True)

# --- 3. 定義語言包 ---
TRANSLATIONS = {
    'zh': {
        'title': "Jose Lab.", 
        'subtitle': "THE LAB bringing clarity and strategy to OPPO’s inventory across Walmart and Bodega.",
        'sec_settings': "參數設定",
        'sec_financial': "價格表",
        'sec_filters': "全域篩選",
        'sec_npi': "NPI 策略",
        'sec_metrics': "PO 概況",
        'target_woi': "目標庫存週數",
        'growth_rate': "預估成長率 (%)",
        'seasonality': "季節係數",
        'force_npi': "NPI 0庫存強制補貨",
        'po_date': "預計 PO 下單日",
        'arrival_info': "🚚 到貨：{}\n(Lead: {} 週)",
        'upload_att': "1. 上傳 AT&T (SO+INV)",
        'upload_inv': "2. 上傳 Telcel (INV)",
        'upload_so': "3. 上傳 Telcel (SO)",
        'upload_prompt': "👋 請上傳 Excel 檔案以開始分析。",
        'loading_data': "數據處理中...",
        'loading_npi': "計算鋪貨邏輯...",
        'week_header': "📅 選擇銷量週次",
        'week_select': "選擇週次",
        'date_header': "📆 選擇銷量日期 (全域)",
        'date_select': "選擇日期 (預設全部)",
        'npi_select_label': "確認 NPI 機型:",
        'npi_warning': "⚠️ 未選擇 NPI 機型。",
        'store_type': "門店類型",
        'type_all': "全部",
        'type_active': "🔥 活躍",
        'type_inactive': "💤 非活躍",
        'total_po_units': "Total PO (台數)",
        'total_po_value': "Total PO (金額)",
        'deur_total': "Deur (1-8)",
        'r9_total': "R9 (9C+9F)",
        'po_breakdown': "📋 PO 機型細項 (Breakdown)",
        'tab1': "🔥 門店詳細",
        'tab2': "🗺️ 區域分析",
        'tab3': "📈 總部匯總",
        'tab_high_end': "💎 高端機分析",
        'tab4': "🧠 AI 超級診斷",
        'tab5': "🔮 未來預測",
        'tab6': "💰 促銷計算",
        'tab7': "✍️ PO 試算表",
        'tab8': "📅 2026 預測",
        'tab9': "🏆 S 級門店 (Top 20%)",
        'tab10': "🏭 門店 CAPA 容量", 
        'fin_set_price': "設定 ASP (平均售價)",
        'plan_checklist': "### 🎯 執行清單",
        'check_npi': "確認 NPI 訂單",
        'check_risk': "下載斷貨清單",
        'check_zombie': "下載呆滯清單",
        'high_end_title': "💎 高端機 vs A系列 結構分析",
        'he_mix_title': "📦 庫存金額佔比",
        'he_sales_title': "💳 銷售台數佔比",
        'he_table_header': "💎 RENO 系列詳細監控",
        'promo_title': "💰 促銷 ROI 計算",
        'promo_summary': "### 💵 財務預覽",
        'ai_no_data': "無資料。",
        'hygiene_title': "數據健康度",
        'hygiene_ok': "數據結構健康",
        'hygiene_bad_model': "發現未知機型: {}",
        'ai_share_sales_title': "📈 銷量全機型佔比 (Sales Share)",
        'ai_share_inv_title': "📦 庫存全機型佔比 (Inventory Share)",
        'ai_loss_title': "📉 NPI 12週斷貨潛在損失 (Cost of Inaction)",
        'ai_loss_desc': "計算公式：(預測12週銷量 - 現有庫存) * ASP。代表如果不補貨，未來一季將流失的市場價值。",
        'ai_reno_top10_title': "💎 Top 10 RENO 銷售冠軍店",
        'ai_all_top10_title': "🔥 Top 10 全機型總銷量冠軍店",
        'future_high_title': "🌟 近期高光 (Top Performers)",
        'future_low_title': "❄️ 近期低光 (Slow Moving Models)",
        'future_risk_title': "⚠️ 未來危機 (Stock Risk Alert)",
        'future_highlight_msg': "機型 **{}** 表現優異，全通路週銷量 **{:.0f}** 台，WOS **{:.1f}** 週 (健康)。",
        'future_lowlight_msg': "機型 **{}** 全通路積壓 **{:.0f}** 台，但週銷僅 **{:.1f}** 台。請考慮降價出清，勿再進貨。",
        'future_risk_msg': "機型 **{}** 買氣強勁 (週銷 {:.0f})，但庫存僅剩 {:.0f} 台，預計 **{:.1f}** 週後斷貨！",
        'future_advice_restock': "🚨 **緊急補貨**：市場缺口巨大，請立即為 **{}** 下單。",
        'promo_total_units': "預計總銷量",
        'promo_total_budget': "總預算",
        'po_manual_title': "✍️ PO 手動試算 (Manual Worksheet)",
        'po_manual_col_model': "機型 (Model)",
        'po_manual_col_telcel': "Telcel DEUR",
        'po_manual_col_att': "ATT",
        'po_manual_col_total': "TOTAL",
        'promo_detail_title': "📊 預測結果詳情 (Detailed Breakdown)",
        'promo_col_promo_price': "促銷後價格",
        'promo_col_spending': "我方支出 (50%)",
        'shop_filter_label': "門店代號 (Shop Number)",
        'clear_cache_btn': "🔄 清除快取 & 重新讀取",
        'promo_ai_advisor_title': "🤖 AI 促銷策略顧問 (AI Promo Advisor)",
        'promo_ai_zombie': "🧟 **殭屍庫存 (Zombie)**：庫存 {:,} 台 / 銷量 0",
        'promo_ai_critical': "🛑 **Crítico (>30 WOS)**: WOS {} sem. ¡Liquidación 30-40%!",
        'promo_ai_slow': "🐢 **Lento (12-30 WOS)**: WOS {} sem. Desc. 10%.",
        'promo_ai_healthy': "🟢 **Saludable (8-12 WOS)**: WOS {} sem. Balanceado.",
        'promo_ai_oos': "🔴 **Riesgo (<8 WOS)**: WOS {} sem. No Promocionar.",
        'promo_act_clearance': "📉 **Acción: Liquidación Agresiva**",
        'promo_act_stimulate': "✨ **Acción: Estimular Venta**",
        'promo_act_bundle': "🎁 **Acción: Bundle / Spiff**",
        'promo_act_hold': "🛡️ **Acción: Mantener Precio**",
        'promo_impact_critical': "⚠️ **Impacto**: Exceso crítico. Se requiere descuento profundo (30-40%) para mover stock.",
        'promo_impact_slow': "💡 **Impacto**: Rotación lenta. Descuento ligero (10%) para reactivar.",
        'promo_impact_do': "✅ **Si actúas**: Recuperas $ {:,.0f} en efectivo y liberas OTB.",
        'promo_impact_dont': "❌ **Si no actúas**: Costo de inventario estancado $ {:,.0f} / mes.",
        'promo_impact_risk': "⚠️ **Alerta**: Stock bajo. Promoción causará quiebre de stock.",
        'heatmap_title': "🗺️ 區域庫存熱力圖 (Inventory Heatmap - Red Alert)",
        'download_report_btn': "📥 下載完整戰略報告 (Excel)",
        'trend_wow_title': "📈 Tendencia Semanal (WoW)",
        'forecast_title': "🔮 2026 全年銷量預測 (分價位段)",
        'forecast_growth_label': "2026 目標增長率 (%)",
        'seg_low': "A LOW",
        'seg_mid': "A MEDIUM",
        'seg_mid_high': "A HIGH",
        'seg_high': "RENO + FIND",
        's_class_title': "🏆 S 級門店戰情室 (Top 20% Contributors)",
        's_class_desc': "這些門店貢獻了絕大部分的業績，必須優先確保供貨與監控。",
        's_class_insight': "⚠️ **注意 (Action Required)**",
        's_col_rank': "排名",
        's_col_insight': "AI 診斷",
        'sec_capa': "CAPA 容量設定",
        'capa_col_region': "區域 (SUB REGION)",
        'capa_col_shop_num': "門店編號 (SHOP NUMBER)",
        'capa_col_shop_name': "門店名稱 (SHOP NAME)",
        'capa_col_capa': "原始 CAPA (Original)",
        'capa_title_main': "🏭 門店容量工具 (CAPA Tool)",
        'capa_season_header': "📅 季節性係數設定 (Seasonality Settings)",
        'capa_month_select': "選擇當前月份 (Select Month)",
        'capa_applied_factor': "應用係數 (Factor)",
        'capa_adj_capa': "調整後 CAPA (Adjusted)",
        'capa_select_none': "不選擇 (None / Default)"
    },
    'es': {
        'title': "Jose Lab.", 
        'subtitle': "THE LAB bringing clarity and strategy to OPPO’s inventory across Walmart and Bodega.",
        'sec_settings': "CONFIGURACIÓN",
        'sec_financial': "PRECIOS",
        'sec_filters': "FILTROS",
        'sec_npi': "ESTRATEGIA NPI",
        'sec_metrics': "RESUMEN PO",
        'target_woi': "Target WOS",
        'growth_rate': "Crecimiento (%)",
        'seasonality': "Factor Estacional",
        'force_npi': "Min Qty NPI (Inv 0)",
        'po_date': "Fecha PO",
        'arrival_info': "LLEGADA: {}\nLEAD TIME: {} SEM",
        'upload_att': "1. Cargar AT&T",
        'upload_inv': "2. Cargar Telcel (INV)",
        'upload_so': "3. Cargar Telcel (SO)",
        'upload_prompt': "Carga los archivos.",
        'loading_data': "PROCESANDO...",
        'loading_npi': "CALCULANDO...",
        'week_header': "Semanas de Venta",
        'week_select': "Semanas",
        'date_header': "📆 Fecha de Venta (Global)",
        'date_select': "Seleccionar Fechas",
        'npi_select_label': "Confirmar NPI:",
        'npi_warning': "Sin NPI seleccionado.",
        'store_type': "Estado Tienda",
        'type_all': "Todas",
        'type_active': "Activas",
        'type_inactive': "Inactivas",
        'total_po_units': "Total PO (Pzs)",
        'total_po_value': "Total PO ($ Value)",
        'deur_total': "Deur",
        'r9_total': "R9",
        'po_breakdown': "📋 Desglose PO",
        'tab1': "Detalle Tienda",
        'tab2': "Regional",
        'tab3': "Resumen HQ",
        'tab_high_end': "Análisis High-End",
        'tab4': "Diagnóstico AI",
        'tab5': "Predicción",
        'tab6': "Calculadora",
        'tab7': "✍️ Hoja PO",
        'tab8': "📅 Pronóstico 2026",
        'tab9': "🏆 Tiendas Clase S",
        'tab10': "🏭 CAPA Tienda", 
        'fin_set_price': "Configurar ASP",
        'plan_checklist': "### CHECKLIST",
        'check_npi': "Confirmar PO NPI",
        'check_risk': "Descargar Lista Riesgo",
        'check_zombie': "Descargar Lista Zombie",
        'high_end_title': "High-End vs Serie A",
        'he_mix_title': "Mix Valor Inventario",
        'he_sales_title': "Mix Venta Unidades",
        'he_table_header': "Monitor Serie Reno",
        'promo_title': "Calculadora ROI",
        'promo_summary': "### RESUMEN FINANCIERO",
        'ai_no_data': "Sin datos.",
        'hygiene_title': "Salud de Datos",
        'hygiene_ok': "Datos Saludables",
        'hygiene_bad_model': "Modelos Desconocidos: {}",
        'ai_share_sales_title': "📈 Share de Venta (Sales Mix)",
        'ai_share_inv_title': "📦 Share de Inventario (Inv Mix)",
        'ai_loss_title': "📉 Costo de Oportunidad NPI (12 Sem)",
        'ai_loss_desc': "Valor de mercado perdido si no se reabastece.",
        'ai_reno_top10_title': "💎 Top 10 Tiendas RENO",
        'ai_all_top10_title': "🔥 Top 10 Tiendas Total",
        'future_high_title': "🌟 Highlights (Top Sales)",
        'future_low_title': "❄️ Lowlights (Stock Pesado / Baja Venta)",
        'future_risk_title': "⚠️ Riesgos (OOS en 4 Sem)",
        'future_highlight_msg': "Modelo **{}** líder. Venta sem **{:.0f}** pzs. WOS **{:.1f}** (Saludable).",
        'future_lowlight_msg': "Modelo **{}** tiene **{:.0f}** pzs pero venta **{:.1f}**. ¡Liquidación requerida!",
        'future_risk_msg': "Modelo **{}** vende bien ({:.0f}) pero stock bajo ({:.0f}). ¡Quiebre en **{:.1f}** sem!",
        'future_advice_restock': "🚨 **Urgente**: Generar PO para **{}** inmediatamente.",
        'promo_total_units': "Unidades Est.",
        'promo_total_budget': "Presupuesto Total",
        'po_manual_title': "✍️ Hoja de Cálculo PO (Manual)",
        'po_manual_col_model': "Modelo",
        'po_manual_col_telcel': "Telcel DEUR",
        'po_manual_col_att': "ATT",
        'po_manual_col_total': "TOTAL",
        'promo_detail_title': "📊 Detalle de Resultados (Breakdown)",
        'promo_col_promo_price': "Precio Promo",
        'promo_col_spending': "Gasto (50%)",
        'shop_filter_label': "ID Tienda (Shop Number)",
        'clear_cache_btn': "🔄 Borrar Caché y Recargar",
        'promo_ai_advisor_title': "🤖 AI Promo Advisor",
        'promo_ai_zombie': "🧟 **Zombie**: Stock {:,} / Venta 0",
        'promo_ai_critical': "🛑 **Crítico (>30 WOS)**: WOS {} sem. ¡Liquidación 30-40%!",
        'promo_ai_slow': "🐢 **Lento (12-30 WOS)**: WOS {} sem. Desc. 10%.",
        'promo_ai_healthy': "🟢 **Saludable (8-12 WOS)**: WOS {} sem. Balanceado.",
        'promo_ai_oos': "🔴 **Riesgo (<8 WOS)**: WOS {} sem. No Promocionar.",
        'promo_act_clearance': "📉 **Acción: Liquidación Agresiva**",
        'promo_act_stimulate': "✨ **Acción: Estimular Venta**",
        'promo_act_bundle': "🎁 **Acción: Bundle / Spiff**",
        'promo_act_hold': "🛡️ **Acción: Mantener Precio**",
        'promo_impact_critical': "⚠️ **Impacto**: Exceso crítico. Se requiere descuento profundo (30-40%) para mover stock.",
        'promo_impact_slow': "💡 **Impacto**: Rotación lenta. Descuento ligero (10%) para reactivar.",
        'promo_impact_do': "✅ **Si actúas**: Recuperas $ {:,.0f} en efectivo y liberas OTB.",
        'promo_impact_dont': "❌ **Si no actúas**: Costo de inventario estancado $ {:,.0f} / mes.",
        'promo_impact_risk': "⚠️ **Alerta**: Stock bajo. Promoción causará quiebre de stock.",
        'heatmap_title': "🗺️ Mapa de Calor de Inventario (Red Alert)",
        'download_report_btn': "📥 Descargar Reporte Completo (Excel)",
        'trend_wow_title': "📈 Tendencia Semanal (WoW)",
        'forecast_title': "🔮 Pronóstico 2026 (Por Segmento de Precio)",
        'forecast_growth_label': "Crecimiento 2026 (%)",
        'seg_low': "A LOW",
        'seg_mid': "A MEDIUM",
        'seg_mid_high': "A HIGH",
        'seg_high': "RENO + FIND",
        's_class_title': "🏆 Tiendas Clase S (Top 20%)",
        's_class_desc': "Estas tiendas generan el 80% de la venta. Prioridad máxima.",
        's_class_insight': "⚠️ **Diagnóstico (Action)**",
        's_col_rank': "#",
        's_col_insight': "AI Insight",
        'sec_capa': "Configuración CAPA",
        'capa_col_region': "Región (SUB REGION)",
        'capa_col_shop_num': "ID Tienda (SHOP NUMBER)",
        'capa_col_shop_name': "Nombre Tienda (SHOP NAME)",
        'capa_col_capa': "CAPA Original",
        'capa_title_main': "🏭 Herramienta de Capacidad (CAPA)",
        'capa_season_header': "📅 Configuración Estacional",
        'capa_month_select': "Seleccionar Mes",
        'capa_applied_factor': "Factor Aplicado",
        'capa_adj_capa': "CAPA Ajustado",
        'capa_select_none': "Ninguno / Default"
    }
}

# --- 4. Language Selector ---
st.sidebar.header("Language / Idioma")
lang_choice = st.sidebar.radio("Select:", ("🇹🇼 繁體中文", "🇲🇽 Español", "🇺🇸 English"), label_visibility="collapsed")

if "繁體中文" in lang_choice: lang = 'zh'
elif "Español" in lang_choice: lang = 'es'
else: lang = 'en'
if lang == 'en': lang = 'es'

def t(key, *args):
    text = TRANSLATIONS[lang].get(key, key)
    if args: return text.format(*args)
    return text

st.title(t('title'))

# --- MEXICAN HOLIDAY TICKER ---
holiday_data = [
    "📅 6 Ene: Día de Reyes (Impact: 1.1x)",
    "💘 14 Feb: San Valentín (Impact: 1.2x)",
    "💐 10 May: Día de las Madres (Impact: 1.5x - High)",
    "🔥 May/Jun: Hot Sale (Impact: 1.8x - Peak)",
    "👔 15 Jun: Día del Padre (Impact: 1.1x)",
    "🎒 Ago: Regreso a Clases (Impact: 1.1x)",
    "🇲🇽 16 Sep: Independencia (Impact: 1.0x)",
    "💀 Oct/Nov: Día de Muertos (Impact: 1.0x)",
    "🛍️ 14-17 Nov: El Buen Fin (Impact: 2.5x - MEGA PEAK)",
    "🎄 Dec: Navidad/Posadas (Impact: 2.0x - High)"
]
ticker_content = "".join([f"<div class='ticker__item'>{h}</div>" for h in holiday_data])

st.markdown(f"""
<div class="ticker-wrap">
<div class="ticker">
    {ticker_content}
</div>
</div>
""", unsafe_allow_html=True)

st.markdown(t('subtitle'))

pd.set_option("styler.render.max_elements", 2000000)

# --- V31.1: Cache Clearing ---
if st.sidebar.button(t('clear_cache_btn')):
    st.cache_data.clear()
    st.rerun()

# --- 5. 檔案上傳區 ---
with st.container():
    c1, c2, c3 = st.columns(3)
    file_att = c1.file_uploader(t('upload_att'), type=['xlsx', 'csv'])
    file_telcel_inv = c2.file_uploader(t('upload_inv'), type=['xlsx', 'csv'])
    file_telcel_so = c3.file_uploader(t('upload_so'), type=['xlsx', 'csv'])

# --- 6. 核心處理函數 ---

def clean_week_format(val):
    s = str(val).strip()
    match = re.search(r'(\d+)', s)
    if match:
        return str(int(match.group(1)))
    return s

def normalize_model_names(df, col='MODELO OPPO'):
    if col in df.columns:
        df[col] = df[col].astype(str).str.upper().str.replace(r'\s+', '', regex=True)
        df = df[~df[col].isin(['NAN', '', 'NONE'])]
    return df

def standardize_columns(df):
    df.columns = df.columns.str.strip().str.upper()
    if 'SEMANA' in df.columns:
        df.rename(columns={'SEMANA': 'WEEK'}, inplace=True)
    
    # 2025-05-20 Update: Improved Date Detection (Aggressive)
    for col in df.columns:
        col_str = str(col).upper()
        if 'FECHA' in col_str or 'DATE' in col_str or 'DIA' in col_str:
            # Avoid renaming 'MEDIA' (AVG) to DATE if misread
            if col_str not in ['MEDIA', 'MEDIAN', 'DIAMETER']:
                df.rename(columns={col: 'DATE'}, inplace=True)
                break # Only rename the first date-like column found

    df = df.loc[:, ~df.columns.duplicated()]
    if 'SUB REGION' in df.columns:
        if 'REGION' in df.columns: df = df.drop(columns=['REGION']) 
        df.rename(columns={'SUB REGION': 'REGION'}, inplace=True)
    elif 'REGION' not in df.columns: df['REGION'] = 'Unknown'
    if 'WEEK' in df.columns: df['WEEK'] = df['WEEK'].apply(clean_week_format)
    return df

def safe_read_csv(file):
    try:
        file.seek(0)
        return pd.read_csv(file)
    except UnicodeDecodeError:
        file.seek(0)
        return pd.read_csv(file, encoding='ISO-8859-1')

@st.cache_data(ttl=3600, show_spinner=False)
def get_all_weeks(files):
    weeks = set()
    for f in files:
        if f is not None:
            try:
                f.seek(0)
                if f.name.endswith('xlsx'): 
                    header = pd.read_excel(f, nrows=1)
                    week_col = None
                    for c in header.columns:
                        c_upper = str(c).upper().strip()
                        if c_upper == 'WEEK' or c_upper == 'SEMANA' or c_upper == 'WK':
                            week_col = c
                            break
                    if week_col:
                        temp = pd.read_excel(f, usecols=[week_col])
                        temp.rename(columns={week_col: 'WEEK'}, inplace=True)
                    else: temp = pd.DataFrame()
                else: 
                    temp = safe_read_csv(f)
                    temp.columns = temp.columns.str.upper()
                    if 'SEMANA' in temp.columns: temp.rename(columns={'SEMANA': 'WEEK'}, inplace=True)
                    if 'WEEK' in temp.columns: temp = temp[['WEEK']]
                
                if 'WEEK' in temp.columns:
                    w_list = temp['WEEK'].dropna().apply(clean_week_format).unique()
                    weeks.update(w_list)
                f.seek(0)
            except: pass
    try: return sorted(list(weeks), key=lambda x: int(x), reverse=True)
    except: return sorted(list(weeks), reverse=True)

def get_latest_week(df):
    if 'WEEK' not in df.columns: return None
    try:
        weeks = pd.to_numeric(df['WEEK'], errors='coerce').dropna()
        if not weeks.empty:
            return str(int(weeks.max()))
        return df['WEEK'].max()
    except: return df['WEEK'].max()

@st.cache_data(ttl=3600, show_spinner=False)
def process_att(file, selected_weeks):
    try: df = pd.read_excel(file)
    except: df = safe_read_csv(file)
    df = standardize_columns(df)
    
    # Force B column (index 1) to be SHOP NUMBER if exists
    if len(df.columns) > 1:
        df['SHOP NUMBER'] = df.iloc[:, 1].astype(str).str.strip()
    
    if 'SHOP NUMBER' in df.columns:
        df['SHOP NUMBER'] = df['SHOP NUMBER'].replace(['nan', 'NaN', 'None', '', '<NA>'], np.nan)
        if 'SHOP NAME' in df.columns:
            df['SHOP NUMBER'] = df['SHOP NUMBER'].fillna(df['SHOP NAME'])
        df['SHOP NUMBER'] = df['SHOP NUMBER'].fillna('Unknown')
    else:
        df['SHOP NUMBER'] = 'Unknown'

    df = normalize_model_names(df)
    df['SHOP SO'] = pd.to_numeric(df['SHOP SO'], errors='coerce').fillna(0)
    df['SHOP INV'] = pd.to_numeric(df['SHOP INV'], errors='coerce').fillna(0)
    
    if 'PROMOTOR' not in df.columns: df['PROMOTOR'] = 'No'
    def check_promotor(val):
        s = str(val).lower().strip()
        if s in ['no', 'nan', '0', '0.0', 'false', 'none', '']: return "No"
        return "Yes"
    df['Has_Promotor'] = df['PROMOTOR'].apply(check_promotor)
 
    if 'TIPO' not in df.columns: df['TIPO'] = 'Regular'
    else: df['TIPO'] = df['TIPO'].astype(str).str.strip().str.upper()

    # =========== ⬇️ 新增這段修正代碼 (FIX START) ⬇️ ===========
    # 邏輯：根據該門店在 Excel 裡「最新一週」的區域，來更新它的歷史區域。
    # 這樣未來你改成 9G 或其他名字，程式都會自動抓最新的名字去覆蓋舊歷史，圖表就不會斷掉。
    
    # 1. 確保週次是數字以便排序
    df['Temp_Week_Num'] = pd.to_numeric(df['WEEK'], errors='coerce')
    
    # 2. 依照週次由新到舊排序，這樣 drop_duplicates 會保留最新的那個區域
    df_sorted = df.sort_values('Temp_Week_Num', ascending=False)
    
    # 3. 建立 {門店: 最新區域} 的對照表
    latest_region_map = df_sorted.drop_duplicates('SHOP NUMBER').set_index('SHOP NUMBER')['REGION'].to_dict()
    
    # 4. 將這個最新區域應用到該門店的所有歷史數據
    df['REGION'] = df['SHOP NUMBER'].map(latest_region_map).fillna(df['REGION'])
    
    # 5. 刪除暫存欄位
    df.drop(columns=['Temp_Week_Num'], inplace=True)
    # =========== ⬆️ 新增這段修正代碼 (FIX END) ⬆️ ===========

    # Granular Trend (這是原本的第 400 行左右)
    trend_att = df.groupby(['WEEK', 'MODELO OPPO', 'REGION', 'SHOP NAME', 'SHOP NUMBER', 'Has_Promotor', 'TIPO'])[['SHOP SO', 'SHOP INV']].sum().reset_index()
# ... (原本的代碼) ...

    # Granular Trend
    trend_att = df.groupby(['WEEK', 'MODELO OPPO', 'REGION', 'SHOP NAME', 'SHOP NUMBER', 'Has_Promotor', 'TIPO'])[['SHOP SO', 'SHOP INV']].sum().reset_index()
    trend_att['Channel'] = 'AT&T'
    trend_att.rename(columns={'REGION': 'SUB REGION'}, inplace=True)

    latest_week = get_latest_week(df)
    df_stock = df[df['WEEK'] == latest_week].groupby(['MODELO OPPO', 'SHOP NAME', 'SHOP NUMBER', 'REGION', 'Has_Promotor', 'TIPO'])['SHOP INV'].sum().reset_index()
    
    df_sales_filtered = df[df['WEEK'].isin(selected_weeks)].copy()
    
    # 2025-05-20 Update: Keep DATE if exists
    cols_to_keep = ['WEEK', 'REGION', 'MODELO OPPO', 'SHOP SO']
    if 'DATE' in df_sales_filtered.columns:
        cols_to_keep.append('DATE')
        # Ensure DATE is datetime
        df_sales_filtered['DATE'] = pd.to_datetime(df_sales_filtered['DATE'], errors='coerce').dt.date

    raw_sales_data = df_sales_filtered[cols_to_keep].copy()
    raw_sales_data['Channel'] = 'AT&T'
    raw_sales_data['SHOP NAME'] = df_sales_filtered['SHOP NAME']
    raw_sales_data['SHOP NUMBER'] = df_sales_filtered['SHOP NUMBER']

    divisor = len(selected_weeks) if len(selected_weeks) > 0 else 1
    df_sales_avg = df_sales_filtered.groupby(['MODELO OPPO', 'SHOP NAME', 'SHOP NUMBER'])['SHOP SO'].sum().reset_index()
    df_sales_avg['Avg_Weekly_Sales'] = df_sales_avg['SHOP SO'] / divisor
    
    final = pd.merge(df_stock, df_sales_avg[['MODELO OPPO', 'SHOP NAME', 'SHOP NUMBER', 'Avg_Weekly_Sales']], on=['MODELO OPPO', 'SHOP NAME', 'SHOP NUMBER'], how='left').fillna(0)
    final['Channel'] = 'AT&T'
    
    return final, raw_sales_data, trend_att

# --- HELPER: Smart Rename for Telcel ---
def smart_rename_telcel(df):
    mapping = {
        'SHOP NUMBER': ['NO. TIENDA', 'TIENDAID', 'STOREID', 'ID_TIENDA', 'POS ID', 'SHOP_NUMBER', 'CLAVE'],
        'SHOP NAME': ['TIENDA', 'NOMBRE', 'DESC TIENDA', 'STORE_NAME', 'NOMBRE_TIENDA', 'DEALER'],
        'SHOP INV': ['INVENTARIO', 'ON HAND', 'STOCK', 'INV', 'OH', 'SOH', 'AVAILABLE'],
        'REGION': ['RGN', 'REGION', 'ZONE', 'ZONA'],
        'MODELO OPPO': ['MATERIAL', 'MODELO', 'SKU', 'PRODUCTO']
    }
    for standard_col, variations in mapping.items():
        if standard_col in df.columns: continue 
        for var in variations:
            if var in df.columns:
                df.rename(columns={var: standard_col}, inplace=True)
                break
            # Fuzzy match for Spanish headers
            for col in df.columns:
                if var in str(col).upper():
                    df.rename(columns={col: standard_col}, inplace=True)
                    break
            if standard_col in df.columns: break
    return df

@st.cache_data(ttl=3600, show_spinner=False)
def process_telcel(file_inv, file_so, selected_weeks):
    if file_inv.name.endswith('xlsx'): df_inv = pd.read_excel(file_inv)
    else: df_inv = safe_read_csv(file_inv)
    if file_so.name.endswith('xlsx'): df_so = pd.read_excel(file_so)
    else: df_so = safe_read_csv(file_so)
    
    df_inv = standardize_columns(df_inv)
    df_so = standardize_columns(df_so) 
    
    # 1. Smart Rename
    df_inv = smart_rename_telcel(df_inv)
    df_so = smart_rename_telcel(df_so)
    
    df_inv = normalize_model_names(df_inv)
    df_so = normalize_model_names(df_so)
    
    # 2. Force Shop Number to String (remove decimals .0)
    def clean_shop_id(val):
        s = str(val).strip()
        if s.endswith('.0'): return s[:-2]
        return s

    if 'SHOP NUMBER' in df_inv.columns:
        df_inv['SHOP NUMBER'] = df_inv['SHOP NUMBER'].apply(clean_shop_id)
    elif 'SHOP NAME' in df_inv.columns: 
        df_inv['SHOP NUMBER'] = df_inv['SHOP NAME']
    else:
        df_inv['SHOP NUMBER'] = 'Unknown'
        
    if 'SHOP NUMBER' in df_so.columns:
        df_so['SHOP NUMBER'] = df_so['SHOP NUMBER'].apply(clean_shop_id)
    else:
        if 'SHOP NAME' in df_so.columns: df_so['SHOP NUMBER'] = df_so['SHOP NAME']
        else: df_so['SHOP NUMBER'] = 'Unknown'

    df_inv['SHOP INV'] = pd.to_numeric(df_inv['SHOP INV'], errors='coerce').fillna(0)
    df_so['SHOP SO'] = pd.to_numeric(df_so['SHOP SO'], errors='coerce').fillna(0)

    # 3. Define Raw Sales Data (Fixing NameError)
    shop_region_map = pd.DataFrame()
    if 'REGION' in df_inv.columns:
        shop_region_map = df_inv[['SHOP NUMBER', 'REGION']].drop_duplicates(subset=['SHOP NUMBER'])
    
    df_so_filtered = df_so[df_so['WEEK'].isin(selected_weeks)].copy()
    
    # 2025-05-20 Update: Keep DATE if exists
    cols_to_keep = ['WEEK', 'REGION', 'MODELO OPPO', 'SHOP SO', 'SHOP NUMBER']
    if 'DATE' in df_so_filtered.columns:
        cols_to_keep.append('DATE')
        # Ensure DATE is datetime
        df_so_filtered['DATE'] = pd.to_datetime(df_so_filtered['DATE'], errors='coerce').dt.date

    # Create raw_sales_data for export/trends
    df_so_with_region = pd.merge(df_so_filtered, shop_region_map, on='SHOP NUMBER', how='left')
    if 'REGION' not in df_so_with_region.columns:
        df_so_with_region['REGION'] = 'Unknown'
    else:
        df_so_with_region['REGION'] = df_so_with_region['REGION'].fillna('Unknown')

    raw_sales_data = df_so_with_region[[c for c in cols_to_keep if c in df_so_with_region.columns]].copy()
    raw_sales_data['Channel'] = 'Telcel'
    if 'SHOP NAME' in df_so.columns:
          # Try to map names if possible
          raw_sales_data['SHOP NAME'] = df_so_filtered['SHOP NAME']
    else:
          raw_sales_data['SHOP NAME'] = raw_sales_data['SHOP NUMBER']
          
    # 4. Granular Trend
    t_inv = df_inv.groupby(['WEEK', 'SHOP NUMBER', 'MODELO OPPO'])['SHOP INV'].sum().reset_index()
    t_so = df_so.groupby(['WEEK', 'SHOP NUMBER', 'MODELO OPPO'])['SHOP SO'].sum().reset_index()
    trend_telcel = pd.merge(t_so, t_inv, on=['WEEK', 'SHOP NUMBER', 'MODELO OPPO'], how='outer').fillna(0)
    
    trend_telcel = pd.merge(trend_telcel, shop_region_map, on='SHOP NUMBER', how='left')
    trend_telcel['REGION'] = trend_telcel['REGION'].fillna('Unknown')
    
    # Fetch Shop Names for Trend
    if 'SHOP NAME' in df_inv.columns:
        shop_name_map = df_inv[['SHOP NUMBER', 'SHOP NAME']].drop_duplicates(subset=['SHOP NUMBER'])
        trend_telcel = pd.merge(trend_telcel, shop_name_map, on='SHOP NUMBER', how='left')
    else:
        trend_telcel['SHOP NAME'] = trend_telcel['SHOP NUMBER']

    trend_telcel['Channel'] = 'Telcel'
    trend_telcel.rename(columns={'REGION': 'SUB REGION'}, inplace=True)

    # 5. Main Processing
    if 'PROMOTOR' not in df_inv.columns: df_inv['PROMOTOR'] = 'No'
    def check_promotor(val):
        s = str(val).lower().strip()
        if s in ['no', 'nan', '0', '0.0', 'false', 'none', '']: return "No"
        return "Yes"
    df_inv['Has_Promotor'] = df_inv['PROMOTOR'].apply(check_promotor)
    if 'TIPO' not in df_inv.columns: df_inv['TIPO'] = 'Regular'
    else: df_inv['TIPO'] = df_inv['TIPO'].astype(str).str.strip().str.upper()
    if 'REGION' not in df_inv.columns: df_inv['REGION'] = 'Unknown'

    latest_week = get_latest_week(df_inv)
    
    inv_cols = ['MODELO OPPO', 'SHOP NUMBER', 'Has_Promotor', 'TIPO']
    if 'SHOP NAME' in df_inv.columns: inv_cols.append('SHOP NAME')
    if 'REGION' in df_inv.columns: inv_cols.append('REGION')
    
    inv_grouped = df_inv[df_inv['WEEK'] == latest_week].groupby(inv_cols)['SHOP INV'].sum().reset_index()
    
    divisor = len(selected_weeks) if len(selected_weeks) > 0 else 1
    so_grouped = df_so_filtered.groupby(['MODELO OPPO', 'SHOP NUMBER'])['SHOP SO'].sum().reset_index()
    so_grouped['Avg_Weekly_Sales'] = so_grouped['SHOP SO'] / divisor

    # Merge
    merged = pd.merge(inv_grouped, so_grouped[['MODELO OPPO', 'SHOP NUMBER', 'Avg_Weekly_Sales']], on=['SHOP NUMBER', 'MODELO OPPO'], how='outer')
    merged['Channel'] = 'Telcel'
    
    # Recover missing info (Region/Name) after outer join
    merged = pd.merge(merged, shop_region_map, on='SHOP NUMBER', how='left', suffixes=('', '_map'))
    if 'REGION' in merged.columns:
        merged['REGION'] = merged['REGION'].fillna(merged['REGION_map']).fillna('Unknown')
    else:
        merged['REGION'] = merged.get('REGION_map', 'Unknown')

    if 'SHOP NAME' not in merged.columns and 'SHOP NAME' in df_inv.columns:
          shop_name_map = df_inv[['SHOP NUMBER', 'SHOP NAME']].drop_duplicates(subset=['SHOP NUMBER'])
          merged = pd.merge(merged, shop_name_map, on='SHOP NUMBER', how='left', suffixes=('', '_n'))
          merged['SHOP NAME'] = merged['SHOP NAME'].fillna(merged['SHOP NAME_n'])

    merged['SHOP INV'] = merged['SHOP INV'].fillna(0)
    merged['Avg_Weekly_Sales'] = merged['Avg_Weekly_Sales'].fillna(0)
    if 'SHOP NAME' in merged.columns: merged['SHOP NAME'] = merged['SHOP NAME'].fillna('Store ' + merged['SHOP NUMBER'].astype(str))
    else: merged['SHOP NAME'] = 'Store ' + merged['SHOP NUMBER'].astype(str)
    
    if 'TIPO' in merged.columns: merged['TIPO'] = merged['TIPO'].fillna('Regular')
    if 'Has_Promotor' in merged.columns: merged['Has_Promotor'] = merged['Has_Promotor'].fillna('No')
    
    final_cols = ['MODELO OPPO', 'SHOP NUMBER', 'SHOP NAME', 'REGION', 'Has_Promotor', 'TIPO', 'SHOP INV', 'Avg_Weekly_Sales', 'Channel']
    for c in final_cols:
        if c not in merged.columns: merged[c] = 0 if c == 'SHOP INV' else 'Unknown'
        
    return merged[final_cols], raw_sales_data, trend_telcel

def expand_npi_opportunities(df, active_npi_list):
    if not active_npi_list: return df
    df_list = []
    for channel in df['Channel'].unique():
        df_channel = df[df['Channel'] == channel].copy()
        df_channel = df_channel.loc[:, ~df_channel.columns.duplicated()]
        
        unique_key = 'SHOP NUMBER' if channel == 'Telcel' else 'SHOP NAME'
        cols_to_select = [unique_key, 'SUB REGION', 'Has_Promotor', 'Channel']
        if 'SHOP NAME' not in cols_to_select: cols_to_select.append('SHOP NAME')
        if 'SHOP NUMBER' not in cols_to_select: cols_to_select.append('SHOP NUMBER')
        
        cols_to_select = [c for c in cols_to_select if c in df_channel.columns]
        stores = df_channel[cols_to_select].drop_duplicates(subset=[unique_key])
        if stores.empty: continue
        
        stores['key'] = 1
        npi_df = pd.DataFrame({'MODELO OPPO': active_npi_list, 'TIPO': 'NPI', 'key': 1})
        full_grid = pd.merge(stores, npi_df, on='key').drop('key', axis=1)
        
        merge_keys = [unique_key, 'MODELO OPPO']
        
        expanded = pd.merge(full_grid, df_channel[[unique_key, 'MODELO OPPO', 'SHOP INV', 'Avg_Weekly_Sales']], on=merge_keys, how='left')
        expanded['SHOP INV'] = expanded['SHOP INV'].fillna(0)
        expanded['Avg_Weekly_Sales'] = expanded['Avg_Weekly_Sales'].fillna(0)
        
        non_npi_df = df_channel[~df_channel['MODELO OPPO'].isin(active_npi_list)]
        final_channel_df = pd.concat([expanded, non_npi_df], ignore_index=True)
        
        agg_dict = {'SHOP INV': 'sum', 'Avg_Weekly_Sales': 'sum'}
        for c in final_channel_df.columns:
            if c not in agg_dict and c not in [unique_key, 'MODELO OPPO']:
                agg_dict[c] = 'first'
                
        final_channel_df = final_channel_df.groupby([unique_key, 'MODELO OPPO']).agg(agg_dict).reset_index()
        df_list.append(final_channel_df)
        
    if not df_list: return df
    return pd.concat(df_list, ignore_index=True)


# --- 7. 主程式邏輯 ---
if file_att and file_telcel_inv and file_telcel_so:
    
    with st.spinner(t('loading_data')):
        # V34.0 Turbo: Cache
        all_available_weeks = get_all_weeks([file_att, file_telcel_so])
        
        # --- Settings ---
        with st.sidebar.expander(t('sec_settings'), expanded=False):
            c1, c2 = st.columns(2)
            target_woi = c1.number_input(t('target_woi'), value=12)
            growth_rate = c2.slider(t('growth_rate'), -50, 50, 0) / 100
            seasonality_factor = st.slider(t('seasonality'), 0.8, 2.0, 1.0, 0.1)
            force_npi_qty = st.number_input(t('force_npi'), value=4)
            po_date = st.date_input(t('po_date'), datetime.date.today())
            lead_time_weeks = 2
            st.info(t('arrival_info', po_date + datetime.timedelta(weeks=lead_time_weeks), lead_time_weeks))
            st.markdown("---")
            st.caption(t('week_header'))
            default_selection = all_available_weeks[:4] if len(all_available_weeks) >= 4 else all_available_weeks
            selected_weeks = st.multiselect(t('week_select'), options=all_available_weeks, default=default_selection)
        
        if not selected_weeks:
            st.stop()

        # V34.0 Turbo: Cache Data Processing
        if 'processed_data' not in st.session_state or st.session_state.get('weeks_id') != str(selected_weeks):
            df_att, raw_sales_att, trend_att = process_att(file_att, selected_weeks) # Unpack 3
            df_telcel, raw_sales_telcel, trend_telcel = process_telcel(file_telcel_inv, file_telcel_so, selected_weeks) # Unpack 3
            
            # Merge Main Data
            cols = ['Channel', 'REGION', 'SHOP NAME', 'SHOP NUMBER', 'MODELO OPPO', 'TIPO', 'SHOP INV', 'Avg_Weekly_Sales', 'Has_Promotor']
            for c in cols:
                if c not in df_att.columns: df_att[c] = None
                if c not in df_telcel.columns: df_telcel[c] = None
            
            df_all = pd.concat([df_att[cols], df_telcel[cols]], ignore_index=True)
            df_all.rename(columns={'REGION': 'SUB REGION'}, inplace=True)
            df_all = df_all.loc[:, ~df_all.columns.duplicated()]
            
            # --- UPDATED: Merge Granular Trends for Session State ---
            all_trends_granular = pd.concat([trend_att, trend_telcel], ignore_index=True)
            all_trends_granular['WEEK_NUM'] = pd.to_numeric(all_trends_granular['WEEK'], errors='coerce')
            all_trends_granular = all_trends_granular.sort_values('WEEK_NUM')
            # --------------------------------------------------------

            st.session_state.processed_data = df_all
            st.session_state.raw_sales_att = raw_sales_att
            st.session_state.raw_sales_telcel = raw_sales_telcel
            st.session_state.all_trends_granular = all_trends_granular # Store Granular Trend
            st.session_state.weeks_id = str(selected_weeks)
            
            # --- CAPA Initialization in State (Independent Data) ---
            if 'capa_data' not in st.session_state:
                st.session_state.capa_data = pd.DataFrame(DEFAULT_CAPA_DATA)
                st.session_state.capa_data['SHOP NUMBER'] = st.session_state.capa_data['SHOP NUMBER'].astype(str).str.strip()

        df_all = st.session_state.processed_data
        raw_sales_att = st.session_state.raw_sales_att
        raw_sales_telcel = st.session_state.raw_sales_telcel
        all_trends_granular = st.session_state.all_trends_granular 
        
        # --- FIX: DEFINE df_raw_sales_all GLOBALLY TO PREVENT NAME ERROR ---
        df_raw_sales_all = pd.concat([raw_sales_att, raw_sales_telcel], ignore_index=True)
        df_raw_sales_all.rename(columns={'REGION': 'SUB REGION'}, inplace=True)
        # -------------------------------------------------------------------
        
        # --- NEW: GLOBAL DATE FILTER ---
        st.sidebar.markdown("---")
        st.sidebar.caption(t('date_header'))
        
        # New Feature: DATA DIAGNOSTIC (Did we find dates?)
        with st.sidebar.expander("🕵️ 數據診斷 (Data Check)", expanded=False):
            if 'DATE' in df_raw_sales_all.columns:
                st.success("✅ DATE column found!")
                date_range = df_raw_sales_all['DATE'].dropna().unique()
                if len(date_range) > 0:
                    st.write(f"Range: {min(date_range)} to {max(date_range)}")
                else:
                    st.warning("Column found but empty.")
            else:
                st.error("❌ No DATE column detected.")
                st.write("Columns found:", list(df_raw_sales_all.columns))
        
        # Get all unique dates from raw sales data
        all_dates = []
        if 'DATE' in df_raw_sales_all.columns:
             # Filter out NaT/None
             valid_dates = df_raw_sales_all['DATE'].dropna().unique()
             # Convert to list and sort
             all_dates = sorted(valid_dates, reverse=True)
        
        # Date Multiselect
        selected_dates = st.sidebar.multiselect(t('date_select'), options=all_dates, default=[])
        
        # If dates selected, update sales data in df_all
        if selected_dates:
             # Filter raw data
             filtered_raw = df_raw_sales_all[df_raw_sales_all['DATE'].isin(selected_dates)]
             
             days_count = len(selected_dates)
             if days_count > 0:
                 new_sales_sum = filtered_raw.groupby(['SHOP NUMBER', 'MODELO OPPO'])['SHOP SO'].sum().reset_index()
                 new_sales_sum.rename(columns={'SHOP SO': 'Total_Period_Sales'}, inplace=True)
                 
                 # Normalize to Weekly Rate
                 # If 1 day selected -> Sales * 7
                 # If 7 days selected -> Sales * 1
                 weekly_factor = 7 / days_count
                 new_sales_sum['Avg_Weekly_Sales'] = new_sales_sum['Total_Period_Sales'] * weekly_factor
                 
                 # Merge back into df_all (replace old Avg_Weekly_Sales)
                 # First drop old column
                 if 'Avg_Weekly_Sales' in df_all.columns:
                      df_all = df_all.drop(columns=['Avg_Weekly_Sales'])
                 
                 df_all = pd.merge(df_all, new_sales_sum[['SHOP NUMBER', 'MODELO OPPO', 'Avg_Weekly_Sales']], 
                                   on=['SHOP NUMBER', 'MODELO OPPO'], how='left')
                 df_all['Avg_Weekly_Sales'] = df_all['Avg_Weekly_Sales'].fillna(0)
                 
                 st.sidebar.success(f"📅 Filtered by {days_count} days. Sales Normalized to Weekly Rate.")
        # -------------------------------

        # Retrieve CAPA data for display
        capa_df = st.session_state.capa_data.copy()

    # --- Data Hygiene ---
    with st.expander(t('hygiene_title'), expanded=False):
        unknown_models = df_all[~df_all['MODELO OPPO'].astype(str).str.contains(r'[A-Z0-9]', regex=True, na=False)]['MODELO OPPO'].unique()
        if len(unknown_models) > 0:
            st.warning(t('hygiene_bad_model', len(unknown_models)))
            st.write(unknown_models)
        else:
            st.success(t('hygiene_ok'))

    # --- Financial Layer ---
    all_models = sorted(df_all['MODELO OPPO'].dropna().unique().tolist())
    DEFAULT_PRICES = {
        'A16': 3899, 'A17': 2999, 'A38': 3499, 'A40': 3799, 'A5': 3799,
        'A57': 5999, 'A58': 3999, 'A58B': 5499, 'A58BDLE': 5499,
        'A5PRO': 5299, 'A5PRO5G': 6499, 'A60': 5699, 'A77': 4999, 
        'A78': 5499, 'A79': 6499, 'A80': 6999, 'RENO11': 8499,
        'RENO125G': 10999, 'RENO12F': 8999, 'RENO13': 13999,
        'RENO135G': 13999, 'RENO13F': 8999, 'RENO14F': 9799, 'RENO7': 6999
    }

    with st.sidebar.expander(t('sec_financial'), expanded=True):
        st.caption(t('fin_set_price'))
        initial_data = []
        for model in all_models:
            clean_model = str(model).upper().replace(" ", "")
            price = DEFAULT_PRICES.get(clean_model, 0.0)
            initial_data.append({'MODELO OPPO': model, 'ASP': float(price)})
        price_data = pd.DataFrame(initial_data)
        edited_prices = st.data_editor(
            price_data,
            column_config={
                "MODELO OPPO": st.column_config.TextColumn(t('fin_col_model'), disabled=True),
                "ASP": st.column_config.NumberColumn(t('fin_col_price'), min_value=0, step=100)
            },
            hide_index=True, use_container_width=True, key='price_editor'
        )
        price_map = dict(zip(edited_prices['MODELO OPPO'], edited_prices['ASP']))

    # --- NPI Logic ---
    with st.sidebar.expander(t('sec_npi'), expanded=False):
        detected_npis = df_all[df_all['TIPO'] == 'NPI']['MODELO OPPO'].unique().tolist()
        default_npi = detected_npis if detected_npis else (all_models[:5] if len(all_models) > 0 else [])
        final_npi_list = st.multiselect(t('npi_select_label'), options=all_models, default=default_npi)
        if not final_npi_list: st.warning(t('npi_warning'))
        st.caption(f"📦 NPI Active Models: {len(final_npi_list)}")
    
    with st.spinner(t('loading_npi')):
        df_all = expand_npi_opportunities(df_all, final_npi_list)
        if final_npi_list:
            df_all.loc[df_all['MODELO OPPO'].isin(final_npi_list), 'TIPO'] = 'NPI'
    
    df_all['ASP'] = df_all['MODELO OPPO'].map(price_map).fillna(0)
    
    # --- Calculations ---
    df_all['Predicted_Sales'] = df_all['Avg_Weekly_Sales'] * (1 + growth_rate) * seasonality_factor
    df_all['WOI'] = df_all.apply(lambda x: x['SHOP INV'] / x['Predicted_Sales'] if x['Predicted_Sales'] > 0 else 99, axis=1)
    
    sales_during_lead_time = df_all['Predicted_Sales'] * lead_time_weeks
    
    def calculate_po(row):
        if row['MODELO OPPO'] not in final_npi_list: return 0
        standard_po = 0
        if row['WOI'] < target_woi:
            standard_po = (target_woi - row['WOI']) * row['Predicted_Sales']
        if row['SHOP INV'] == 0:
            return max(standard_po, force_npi_qty)
        return max(standard_po, 0)

    df_all['Suggested_PO_Qty'] = df_all.apply(calculate_po, axis=1).round(0)
    df_all['Suggested_PO_Value'] = df_all['Suggested_PO_Qty'] * df_all['ASP']
    df_all['Stock_in_2w_NO_PO'] = df_all['SHOP INV'] - sales_during_lead_time

    def get_store_activity(row):
        if row['SHOP INV'] > 0 or row['Predicted_Sales'] > 0: return 'Active'
        else: return 'Inactive'
    df_all['Store_Activity'] = df_all.apply(get_store_activity, axis=1)

    # --- Binary Region Logic (R9 vs Deur) ---
    def get_region_group(val):
        s = str(val).upper().strip()
        if '9C' in s or '9F' in s:
            return 'R9 (9C+9F)'
        return 'Deur (1-8)'
    df_all['Region_Group'] = df_all['SUB REGION'].astype(str).apply(get_region_group)

    # --- Filters ---
    st.sidebar.divider()
    st.sidebar.subheader(t('sec_filters'))
    activity_filter = st.sidebar.radio(t('store_type'), [t('type_all'), t('type_active'), t('type_inactive')], index=0)
    channel_options = df_all['Channel'].unique()
    channel_filter = st.sidebar.multiselect(t('channel'), options=channel_options, default=channel_options)
    
    # Shop Filter
    all_shops = sorted(df_all['SHOP NUMBER'].astype(str).unique())
    shop_filter = st.sidebar.multiselect(t('shop_filter_label'), options=all_shops)
    
    all_regions = sorted(df_all['SUB REGION'].astype(str).unique())
    agree_all_regions = st.sidebar.checkbox(t('region_all'), value=True)
    if agree_all_regions: region_filter = all_regions
    else: region_filter = st.sidebar.multiselect(t('region'), options=all_regions, default=[])
    model_options = sorted([x for x in df_all['MODELO OPPO'].unique() if str(x).lower() != 'nan'])
    agree_all_models = st.sidebar.checkbox(t('model_all'), value=True)
    if agree_all_models: model_filter = model_options
    else: model_filter = st.sidebar.multiselect(t('model'), options=model_options, default=[])

    df_calc = df_all.copy()
    if t('type_active') in activity_filter: df_calc = df_calc[df_calc['Store_Activity'] == 'Active']
    elif t('type_inactive') in activity_filter: df_calc = df_calc[df_calc['Store_Activity'] == 'Inactive']
    if channel_filter: df_calc = df_calc[df_calc['Channel'].isin(channel_filter)]
    
    # Apply Shop Filter
    if shop_filter:
        df_calc = df_calc[df_calc['SHOP NUMBER'].astype(str).isin(shop_filter)]
        
    if region_filter: df_calc = df_calc[df_calc['SUB REGION'].astype(str).isin(region_filter)]
    else: df_calc = df_calc[df_calc['SUB REGION'].astype(str).isin([])]
    if model_filter: df_calc = df_calc[df_calc['MODELO OPPO'].isin(model_filter)]
    else: df_calc = df_calc[df_calc['MODELO OPPO'].isin([])]

    # --- High-End vs A-Series Logic ---
    def get_model_tier(val):
        s = str(val).upper().strip()
        if s.startswith('RENO'): return 'High-End'
        if s.startswith('A'): return 'A-Series'
        return 'Other'
    df_calc['Model_Tier'] = df_calc['MODELO OPPO'].apply(get_model_tier)

    # --- Metrics (Unchanged) ---
    grand_total = int(df_calc['Suggested_PO_Qty'].sum())
    grand_total_val = df_calc['Suggested_PO_Value'].sum()
    df_deur = df_calc[df_calc['Region_Group'] == 'Deur (1-8)']
    df_r9 = df_calc[df_calc['Region_Group'] == 'R9 (9C+9F)']
    
    st.sidebar.divider()
    st.sidebar.header(t('sec_metrics'))
    st.sidebar.metric(t('total_po_units'), f"{grand_total:,}")
    st.sidebar.metric(t('total_po_value'), f"$ {grand_total_val:,.0f}")
    c1, c2 = st.sidebar.columns(2)
    c1.metric("Deur", f"{int(df_deur['Suggested_PO_Qty'].sum()):,}")
    c2.metric("R9", f"{int(df_r9['Suggested_PO_Qty'].sum()):,}")

    st.sidebar.markdown("---")
    with st.sidebar.expander(t('po_breakdown'), expanded=True):
        tab_deur, tab_r9 = st.tabs(["Deur", "R9"])
        with tab_deur:
            po_deur = df_deur.groupby('MODELO OPPO')['Suggested_PO_Qty'].sum().reset_index()
            po_deur = po_deur[po_deur['Suggested_PO_Qty'] > 0].sort_values('Suggested_PO_Qty', ascending=False)
            st.dataframe(po_deur.style.format({'Suggested_PO_Qty': '{:,.0f}'}), hide_index=True, use_container_width=True)
        with tab_r9:
            po_r9 = df_r9.groupby('MODELO OPPO')['Suggested_PO_Qty'].sum().reset_index()
            po_r9 = po_r9[po_r9['Suggested_PO_Qty'] > 0].sort_values('Suggested_PO_Qty', ascending=False)
            st.dataframe(po_r9.style.format({'Suggested_PO_Qty': '{:,.0f}'}), hide_index=True, use_container_width=True)

    # --- Tabs ---
    tabs = st.tabs([t('tab1'), t('tab2'), t('tab3'), t('tab_high_end'), t('tab4'), t('tab5'), t('tab6'), t('tab7'), t('tab8'), t('tab9'), t('tab10')]) # New Tab 10

    with tabs[0]: # Detail (WoW Added)
        st.subheader(t('tab1'))
        df_view = df_calc.copy()
        
        # V34.0 WoW for Detail View (unchanged logic)
        if not df_raw_sales_all.empty:
            latest_wk = get_latest_week(df_raw_sales_all)
            if latest_wk and str(latest_wk).isdigit():
                prev_wk = str(int(latest_wk) - 1)
                sales_trend_shop = df_raw_sales_all[df_raw_sales_all['WEEK'].astype(str).isin([latest_wk, prev_wk])].groupby(['SHOP NUMBER', 'MODELO OPPO', 'WEEK'])['SHOP SO'].sum().unstack(fill_value=0)
                if latest_wk in sales_trend_shop.columns and prev_wk in sales_trend_shop.columns:
                    sales_trend_shop['WoW %'] = ((sales_trend_shop[latest_wk] - sales_trend_shop[prev_wk]) / (sales_trend_shop[prev_wk] + 0.1) * 100)
                    df_view = pd.merge(df_view, sales_trend_shop[['WoW %']], on=['SHOP NUMBER', 'MODELO OPPO'], how='left')

        c1, c2 = st.columns(2)
        promotor_filter = c1.checkbox(t('promotor'), value=True)
        npi_only_filter = c2.checkbox(t('show_npi'), value=True)
        if promotor_filter: df_view = df_view[df_view['Has_Promotor'] == 'Yes']
        if npi_only_filter: df_view = df_view[df_view['TIPO'] == 'NPI']
        
        def get_risk_status(row):
            if row['TIPO'] == 'EOL': return t('risk_eol')
            if row['Stock_in_2w_NO_PO'] < 0: return t('risk_oos')
            if row['Stock_in_2w_NO_PO'] < 10: return t('risk_low')
            return t('risk_safe')
        df_view['Status'] = df_view.apply(get_risk_status, axis=1)
        
        def highlight_status(val):
            s = str(val)
            if any(x in s for x in ['risk_oos', 'OOS', 'Agotado', '斷貨']):
                return 'background-color: #FF5252; color: white; font-weight: bold;'
            elif any(x in s for x in ['risk_low', 'Low', 'Bajo', '危險']):
                return 'background-color: #FFD700; color: black; font-weight: bold;'
            elif 'EOL' in s:
                return 'color: #999999;'
            return ''
            
        def trend_icon_sm(val):
            if pd.isna(val): return ""
            if val > 20: return f"🔥"
            if val < -20: return f"❄️"
            return f"{val:.0f}%"

        if 'WoW %' in df_view.columns:
            df_view['Trend'] = df_view['WoW %'].apply(trend_icon_sm)
            cols_to_show = ['SHOP NUMBER', 'Channel', 'SHOP NAME', 'SUB REGION', 'Store_Activity', 'MODELO OPPO', 'TIPO', 'SHOP INV', 'Predicted_Sales', 'Trend', 'Status', 'Suggested_PO_Qty'] 
        else:
            cols_to_show = ['SHOP NUMBER', 'Channel', 'SHOP NAME', 'SUB REGION', 'Store_Activity', 'MODELO OPPO', 'TIPO', 'SHOP INV', 'Predicted_Sales', 'Status', 'Suggested_PO_Qty'] 

        st.dataframe(
            df_view[cols_to_show]
            .sort_values(by=['TIPO', 'Suggested_PO_Qty'], ascending=[False, False])
            .style.format({
                'SHOP INV': '{:.0f}', 
                'Predicted_Sales': '{:.1f}', 
                'Suggested_PO_Qty': '{:.0f}'
            })
            .applymap(highlight_status, subset=['Status']),
            use_container_width=True,
            hide_index=True
        )

    with tabs[1]: # Regional (V34.0 Heatmap Added)
        st.subheader(t('tab2'))
        
        # V34.0 Heatmap
        st.markdown(f"### {t('heatmap_title')}")
        if not df_calc.empty:
            heatmap_data = df_calc.groupby(['SUB REGION', 'MODELO OPPO'])['SHOP INV'].sum().reset_index()
            fig_heat = px.density_heatmap(
                heatmap_data, 
                x='SUB REGION', 
                y='MODELO OPPO', 
                z='SHOP INV', 
                color_continuous_scale='Reds', # V34.2 Requested Red
                title='Inventory Heatmap'
            )
            st.plotly_chart(fig_heat, use_container_width=True)
        
        region_pivot = df_calc.groupby(['Channel', 'SUB REGION', 'Region_Group', 'MODELO OPPO']).agg({
            'SHOP INV': 'sum', 'Predicted_Sales': 'sum', 'Suggested_PO_Qty': 'sum'
        }).reset_index()
        region_pivot['Region_WOI'] = region_pivot.apply(lambda x: x['SHOP INV'] / x['Predicted_Sales'] if x['Predicted_Sales'] > 0 else 99, axis=1)
        st.dataframe(region_pivot.style.format({'SHOP INV': '{:,.0f}', 'Predicted_Sales': '{:.1f}', 'Suggested_PO_Qty': '{:.0f}', 'Region_WOI': '{:.1f}'}).background_gradient(subset=['Region_WOI'], cmap='Greys_r', vmin=4, vmax=16), use_container_width=True)
        
        st.markdown(t('trend_title'))
        if not df_raw_sales_all.empty:
            trend_pivot = df_raw_sales_all.groupby(['MODELO OPPO', 'WEEK'])['SHOP SO'].sum().unstack(fill_value=0)
            try:
                sorted_cols = sorted(trend_pivot.columns, key=lambda x: int(x))
                trend_pivot = trend_pivot[sorted_cols]
            except: pass
            trend_pivot['Total'] = trend_pivot.sum(axis=1)
            trend_pivot = trend_pivot.sort_values('Total', ascending=False)
            st.dataframe(trend_pivot.style.format("{:,.0f}").background_gradient(axis=1, cmap="Greys", subset=trend_pivot.columns[:-1]), use_container_width=True)

    with tabs[2]: # HQ Summary (V34.0 WoW Trend)
        st.subheader(t('tab3'))

        # --- UPDATED: VISUALIZATION (SALES & INVENTORY TREND WITH GLOBAL FILTERS) ---
        if 'all_trends_granular' in st.session_state and not st.session_state.all_trends_granular.empty:
            
            # 1. Get the granular trend data
            trend_filtered = st.session_state.all_trends_granular.copy()
            
            # 2. Apply Global Filters (Same logic as df_calc)
            if shop_filter:
                trend_filtered = trend_filtered[trend_filtered['SHOP NUMBER'].astype(str).isin(shop_filter)]
            if region_filter:
                trend_filtered = trend_filtered[trend_filtered['SUB REGION'].astype(str).isin(region_filter)]
            if model_filter:
                trend_filtered = trend_filtered[trend_filtered['MODELO OPPO'].isin(model_filter)]
            if channel_filter:
                trend_filtered = trend_filtered[trend_filtered['Channel'].isin(channel_filter)]
            
            # 3. Aggregate by Week/Channel for Plotting
            trend_agg = trend_filtered.groupby(['WEEK', 'Channel'])[['SHOP SO', 'SHOP INV']].sum().reset_index()
            
            # 4. Sort by Week Number for correct X-axis order
            trend_agg['WEEK_NUM'] = pd.to_numeric(trend_agg['WEEK'], errors='coerce')
            trend_agg = trend_agg.sort_values('WEEK_NUM')

            st.markdown("### 📊 總部匯總 (Trends - Filtered)")
            c_trend1, c_trend2 = st.columns(2)
            
            # Chart 1: Sales Trend
            with c_trend1:
                st.markdown("**銷量趨勢 (Sales)**")
                if not trend_agg.empty:
                    fig_so = px.line(trend_agg, x='WEEK', y='SHOP SO', color='Channel', markers=True, 
                                     color_discrete_map={'AT&T': '#003366', 'Telcel': '#87CEEB'}) # Dark Blue / Light Blue
                    st.plotly_chart(fig_so, use_container_width=True)
                else:
                    st.info("No Data for Chart")
            
            # Chart 2: Inventory Trend
            with c_trend2:
                st.markdown("**庫存趨勢 (Inventory)**")
                if not trend_agg.empty:
                    fig_inv = px.line(trend_agg, x='WEEK', y='SHOP INV', color='Channel', markers=True,
                                     color_discrete_map={'AT&T': '#003366', 'Telcel': '#87CEEB'})
                    st.plotly_chart(fig_inv, use_container_width=True)
                else:
                    st.info("No Data for Chart")
            st.divider()

            # --- NEW: Daily Sales Trend Chart ---
            if 'DATE' in df_raw_sales_all.columns:
                st.markdown("### 📅 日銷量趨勢 (Daily Sales Trend)")
                daily_trend = df_raw_sales_all.groupby(['DATE', 'Channel'])['SHOP SO'].sum().reset_index()
                if not daily_trend.empty:
                    fig_daily = px.line(daily_trend, x='DATE', y='SHOP SO', color='Channel', markers=True,
                                        title='Daily Sales (Aggregated)',
                                        color_discrete_map={'AT&T': '#003366', 'Telcel': '#87CEEB'})
                    st.plotly_chart(fig_daily, use_container_width=True)
                else:
                    st.info("No daily data available after filtering.")
        # ----------------------------------------------------------------------------

        # --- NEW: TOP 15 STORES WITH REGION COLORS (Unchanged logic) ---
        st.markdown("### 🏆 Top 15 門店排名 (Store Rankings)")
        c_rank1, c_rank2 = st.columns(2)
        
        with c_rank1:
            st.markdown("#### 💎 Top 15 RENO 銷量門店")
            df_reno = df_calc[df_calc['MODELO OPPO'].str.upper().str.contains("RENO", na=False)]
            
            if not df_reno.empty:
                # Group by Name AND Region to keep region data
                top_reno = df_reno.groupby(['SHOP NAME', 'SUB REGION'])['Avg_Weekly_Sales'].sum().reset_index()
                top_reno = top_reno.nlargest(15, 'Avg_Weekly_Sales')
                # Create a combined label
                top_reno['Store_Label'] = top_reno['SUB REGION'].astype(str) + " | " + top_reno['SHOP NAME']
                
                fig_reno = px.bar(top_reno, x='Avg_Weekly_Sales', y='Store_Label', 
                                 color='SUB REGION', # Color by region
                                 orientation='h', text_auto='.1f', 
                                 title="Reno Weekly Sales")
                fig_reno.update_layout(yaxis={'categoryorder':'total ascending'})
                st.plotly_chart(fig_reno, use_container_width=True)
            else:
                st.info("No Reno sales data.")

        with c_rank2:
            st.markdown("#### 🔥 Top 15 總銷量門店 (Total)")
            # Group by Name AND Region
            top_total = df_calc.groupby(['SHOP NAME', 'SUB REGION'])['Avg_Weekly_Sales'].sum().reset_index()
            top_total = top_total.nlargest(15, 'Avg_Weekly_Sales')
            # Create a combined label
            top_total['Store_Label'] = top_total['SUB REGION'].astype(str) + " | " + top_total['SHOP NAME']

            if not top_total.empty:
                fig_total = px.bar(top_total, x='Avg_Weekly_Sales', y='Store_Label', 
                                 color='SUB REGION', # Color by region
                                 orientation='h', text_auto='.1f', 
                                 title="Total Weekly Sales")
                fig_total.update_layout(yaxis={'categoryorder':'total ascending'})
                st.plotly_chart(fig_total, use_container_width=True)
            else:
                st.info("No sales data.")
        
        st.divider()
        # ----------------------------------------------------------------------------

        pivot = df_calc.groupby(['Channel', 'TIPO', 'MODELO OPPO']).agg({
            'SHOP INV': 'sum', 'Predicted_Sales': 'sum', 'Suggested_PO_Qty': 'sum', 'Suggested_PO_Value': 'sum'
        }).reset_index()
        
        # V34.0 WoW Calculation (unchanged logic)
        if not df_raw_sales_all.empty:
            latest_wk = get_latest_week(df_raw_sales_all)
            if latest_wk and str(latest_wk).isdigit():
                prev_wk = str(int(latest_wk) - 1)
                sales_trend = df_raw_sales_all[df_raw_sales_all['WEEK'].astype(str).isin([latest_wk, prev_wk])].groupby(['MODELO OPPO', 'WEEK'])['SHOP SO'].sum().unstack(fill_value=0)
                if latest_wk in sales_trend.columns and prev_wk in sales_trend.columns:
                    sales_trend['WoW %'] = ((sales_trend[latest_wk] - sales_trend[prev_wk]) / (sales_trend[prev_wk] + 0.1) * 100)
                    pivot = pd.merge(pivot, sales_trend[['WoW %']], on='MODELO OPPO', how='left')

        pivot['WOI'] = pivot['SHOP INV'] / pivot['Predicted_Sales']
        
        def trend_icon(val):
            if pd.isna(val): return ""
            if val > 20: return f"🔥 +{val:.1f}%"
            if val < -20: return f"❄️ {val:.1f}%"
            return f"{val:.1f}%"

        if 'WoW %' in pivot.columns:
            pivot['Trend'] = pivot['WoW %'].apply(trend_icon)
            cols = ['Channel', 'TIPO', 'MODELO OPPO', 'SHOP INV', 'Predicted_Sales', 'Suggested_PO_Qty', 'Suggested_PO_Value', 'WOI', 'Trend']
        else:
            cols = pivot.columns

        st.dataframe(pivot[cols].style.format({'SHOP INV': '{:,.0f}', 'Predicted_Sales': '{:.1f}', 'Suggested_PO_Qty': '{:.0f}', 'Suggested_PO_Value': '$ {:,.0f}', 'WOI': '{:.1f}'}).background_gradient(subset=['WOI'], cmap='Greys_r', vmin=4, vmax=16), use_container_width=True)

    with tabs[3]: # High-End Analysis (Unchanged logic)
        st.subheader(t('high_end_title'))
        
        if df_calc.empty:
            st.warning(t('ai_no_data'))
        else:
            tier_stats = df_calc.groupby('Model_Tier').agg({'SHOP INV': 'sum', 'Predicted_Sales': 'sum', 'Suggested_PO_Qty': 'sum'}).reset_index()
            df_calc['Inv_Value'] = df_calc['SHOP INV'] * df_calc['ASP']
            df_calc['Sales_Value'] = df_calc['Predicted_Sales'] * df_calc['ASP']
            tier_value_stats = df_calc.groupby('Model_Tier').agg({'Inv_Value': 'sum', 'Sales_Value': 'sum'}).reset_index()
            
            c1, c2 = st.columns(2)
            with c1:
                st.markdown(t('he_mix_title'))
                if not tier_value_stats.empty:
                    st.bar_chart(tier_value_stats.set_index('Model_Tier')['Inv_Value'], color='#000000')
            with c2:
                st.markdown(t('he_sales_title'))
                if not tier_stats.empty:
                    st.bar_chart(tier_stats.set_index('Model_Tier')['Predicted_Sales'], color='#999999')

            st.divider()
            st.markdown(t('he_table_header'))
            reno_df = df_calc[df_calc['Model_Tier'] == 'High-End'].groupby('MODELO OPPO').agg({'SHOP INV': 'sum', 'Predicted_Sales': 'sum', 'Suggested_PO_Qty': 'sum', 'Suggested_PO_Value': 'sum'}).reset_index()
            if not reno_df.empty:
                reno_df['WOI'] = reno_df['SHOP INV'] / reno_df['Predicted_Sales']
                reno_df = reno_df.sort_values('Suggested_PO_Value', ascending=False)
                st.dataframe(reno_df.style.format({'SHOP INV': '{:,.0f}', 'Predicted_Sales': '{:.1f}', 'Suggested_PO_Qty': '{:,.0f}', 'Suggested_PO_Value': '$ {:,.0f}', 'WOI': '{:.1f}'}).background_gradient(subset=['WOI'], cmap='Greys_r', vmin=4, vmax=16), use_container_width=True)
            else:
                st.info("No High-End (RENO) models found in current selection.")

    with tabs[4]: # AI Powerhouse (V30.6) (Unchanged logic)
        st.subheader(t('tab4'))
        
        if not df_calc.empty:
            c1, c2 = st.columns(2)
            with c1:
                st.markdown(f"### {t('ai_share_sales_title')}")
                so_share = df_calc.groupby('MODELO OPPO')['Predicted_Sales'].sum().reset_index()
                fig_sales = px.pie(so_share, values='Predicted_Sales', names='MODELO OPPO', color_discrete_sequence=px.colors.sequential.Greys_r)
                st.plotly_chart(fig_sales, use_container_width=True)
            with c2:
                st.markdown(f"### {t('ai_share_inv_title')}")
                inv_share = df_calc.groupby('MODELO OPPO')['SHOP INV'].sum().reset_index()
                fig_inv = px.pie(inv_share, values='SHOP INV', names='MODELO OPPO', color_discrete_sequence=px.colors.sequential.Greys_r)
                st.plotly_chart(fig_inv, use_container_width=True)
            
            st.divider()
            st.markdown(f"### {t('ai_loss_title')}")
            st.caption(t('ai_loss_desc'))
            npi_df = df_calc[df_calc['TIPO'] == 'NPI'].copy()
            npi_df['Gap_12w'] = (npi_df['Predicted_Sales'] * 12 - npi_df['SHOP INV']).clip(lower=0)
            npi_df['Loss_Val'] = npi_df['Gap_12w'] * npi_df['ASP']
            total_loss = npi_df['Loss_Val'].sum()
            c2.metric("Total Opportunity Cost", f"$ {total_loss:,.0f}", delta="Risk", delta_color="inverse")
            if not npi_df.empty and total_loss > 0:
                loss_by_model = npi_df.groupby('MODELO OPPO')['Loss_Val'].sum().sort_values(ascending=False)
                st.bar_chart(loss_by_model, color='#000000')

    with tabs[5]: # Future & Strategy (V34.0 Download Button) (Unchanged logic)
        st.subheader(t('tab5'))
        
        # V34.0 Feature: Download Report Button
        st.markdown(f"### {t('download_report_btn')}")
        output = io.BytesIO()
        with pd.ExcelWriter(output, engine='openpyxl') as writer:
            df_calc.to_excel(writer, sheet_name='Full_Data', index=False)
            
            model_stats = df_calc.groupby('MODELO OPPO').agg({'SHOP INV': 'sum', 'Predicted_Sales': 'sum', 'ASP': 'max'}).reset_index()
            model_stats['WOS'] = np.where(model_stats['Predicted_Sales'] > 0, model_stats['SHOP INV'] / model_stats['Predicted_Sales'], 999)
            risks = model_stats[(model_stats['SHOP INV'] - (model_stats['Predicted_Sales'] * 4) < 0) & (model_stats['Predicted_Sales'] > 10) & (model_stats['WOS'] < 4)]
            risks.to_excel(writer, sheet_name='Urgent_Restock', index=False)
            
            zombies = df_calc[(df_calc['SHOP INV'] >= 5) & (df_calc['Predicted_Sales'] == 0)]
            zombies.to_excel(writer, sheet_name='Zombie_List', index=False)
            
        st.download_button(
            label=t('download_report_btn'),
            data=output.getvalue(),
            file_name=f"KAM_Report_{datetime.date.today()}.xlsx",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        )

        st.divider()
        
        # AI Promo Advisor
        st.markdown(f"### {t('promo_ai_advisor_title')}")
        heavy_stock_models = df_calc.groupby('MODELO OPPO')['SHOP INV'].sum()
        heavy_stock_list = heavy_stock_models[heavy_stock_models > 100].index.tolist()
        target_models = list(set(final_npi_list + heavy_stock_list))
        target_models.sort(key=lambda x: heavy_stock_models.get(x, 0), reverse=True)

        st.info(f"🤖 Analyzing {len(target_models)} models")

        for model in target_models:
            model_data = df_calc[df_calc['MODELO OPPO'] == model]
            total_inv = model_data['SHOP INV'].sum()
            total_sales = model_data['Predicted_Sales'].sum()
            wos = total_inv / total_sales if total_sales > 0 else 999
            
            with st.expander(f"🔎 **{model}** - Stock: {int(total_inv)} - WOS: {int(wos)}"):
                if total_sales == 0 and total_inv > 0:
                    st.error(t('promo_ai_zombie', total_inv))
                    st.markdown(t('promo_act_bundle'))
                elif wos > 30:
                    st.error(t('promo_ai_critical', int(wos)))
                    st.markdown(t('promo_act_clearance'))
                elif wos > 12:
                    st.warning(t('promo_ai_slow', int(wos))) 
                    st.markdown(t('promo_act_stimulate'))
                elif wos < 8:
                    st.error(t('promo_ai_oos', int(wos)))
                    st.markdown(t('promo_act_hold'))
                else:
                    st.success(t('promo_ai_healthy', int(wos)))
                    st.markdown("✅ **Action: Maintain**")

    with tabs[6]: # Promo (Unchanged logic)
        st.subheader(t('promo_title'))
        c1, c2 = st.columns(2)
        promo_start = c1.date_input("Start", datetime.date.today())
        promo_end = c2.date_input("End", datetime.date.today() + datetime.timedelta(weeks=4))
        promo_models = st.multiselect(t('promo_models_label'), options=model_options, default=final_npi_list)
        if promo_models:
            base_run_rate = df_calc[df_calc['MODELO OPPO'].isin(promo_models)].groupby('MODELO OPPO').agg({'Predicted_Sales': 'sum', 'SHOP INV': 'sum', 'ASP': 'max'}).reset_index()
            base_run_rate.rename(columns={'Predicted_Sales': 'Weekly_Velocity', 'ASP': 'Original_Price'}, inplace=True)
            base_run_rate['Promo_Depth_%'] = 0.0
            edited_df = st.data_editor(base_run_rate, column_config={"Original_Price": st.column_config.NumberColumn(disabled=True), "Promo_Depth_%": st.column_config.NumberColumn(min_value=0, max_value=100, step=1)}, hide_index=True, use_container_width=True)
            if not edited_df.empty:
                duration_weeks = (promo_end - promo_start).days / 7
                edited_df['Forecast_SO'] = edited_df['Weekly_Velocity'] * duration_weeks * (1 + edited_df['Promo_Depth_%']/100)  
                contribution_factor = 0.5  
                edited_df['Total_Spending'] = edited_df['Forecast_SO'] * edited_df['Original_Price'] * (edited_df['Promo_Depth_%']/100) * contribution_factor
                edited_df['Promo_Price'] = edited_df['Original_Price'] * (1 - edited_df['Promo_Depth_%']/100)
                st.markdown("---")
                m1, m2 = st.columns(2)
                m1.metric("Total Forecast Units", f"{int(edited_df['Forecast_SO'].sum()):,}")
                m2.metric("Total Promo Budget (50%)", f"$ {edited_df['Total_Spending'].sum():,.0f}")
                st.divider()
                breakdown_df = edited_df[['MODELO OPPO', 'Original_Price', 'Promo_Depth_%', 'Promo_Price', 'Forecast_SO', 'Total_Spending']].copy()
                st.dataframe(breakdown_df.style.format({'Original_Price': '$ {:,.0f}', 'Promo_Depth_%': '{:.0f}%', 'Promo_Price': '$ {:,.0f}', 'Forecast_SO': '{:,.0f}', 'Total_Spending': '$ {:,.0f}'}), use_container_width=True, column_config={"Promo_Price": t('promo_col_promo_price'), "Forecast_SO": t('promo_col_forecast'), "Total_Spending": t('promo_col_spending')})

    with tabs[7]: # PO Worksheet (Unchanged logic)
        st.subheader(t('po_manual_title'))
        if 'po_worksheet_data' not in st.session_state:
            models_for_po = ["A5", "A5 pro 4G", "A5 pro 5G", "Reno 14 F", "A6x", "A6 pro 5G", "FIND X9 PRO"]
            st.session_state.po_worksheet_data = pd.DataFrame({
                "MODELO": models_for_po,
                "Telcel DEUR": [0] * len(models_for_po),
                "ATT": [0] * len(models_for_po)
            })
        edited_po = st.data_editor(st.session_state.po_worksheet_data, column_config={"MODELO": st.column_config.TextColumn(t('po_manual_col_model'), disabled=True), "Telcel DEUR": st.column_config.NumberColumn(t('po_manual_col_telcel'), min_value=0, step=10), "ATT": st.column_config.NumberColumn(t('po_manual_col_att'), min_value=0, step=1)}, hide_index=True, use_container_width=True, key="po_editor_widget")
        st.session_state.po_worksheet_data = edited_po
        display_df = edited_po.copy()
        display_df["TOTAL"] = display_df["Telcel DEUR"] + display_df["ATT"]
        total_telcel = display_df["Telcel DEUR"].sum()
        total_att = display_df["ATT"].sum()
        total_grand = display_df["TOTAL"].sum()
        st.divider()
        c1, c2, c3 = st.columns(3)
        c1.metric("Telcel Total", f"{total_telcel:,}")
        c2.metric("ATT Total", f"{total_att:,}")
        c3.metric("Grand Total", f"{total_grand:,}")
        st.markdown("#### 📊 計算結果 (Calculated View)")
        st.dataframe(display_df.style.format("{:,}", subset=["Telcel DEUR", "ATT", "TOTAL"]), use_container_width=True, column_config={"TOTAL": t('po_manual_col_total')})

    with tabs[8]: # NEW: TAB 8 (2026 Forecast with Editable Seasonality) (Unchanged logic)
        st.subheader(t('forecast_title'))
        
        c_set1, c_set2 = st.columns([1, 2])
        
        with c_set1:
            st.markdown("### 1. 增長設定")
            growth_2026 = st.slider(t('forecast_growth_label'), -50, 100, 10, format="%d%%") / 100.0

        if 'all_trends_granular' in st.session_state:
            forecast_base = st.session_state.all_trends_granular.copy()
            
            # Global Filters
            if shop_filter: forecast_base = forecast_base[forecast_base['SHOP NUMBER'].astype(str).isin(shop_filter)]
            if region_filter: forecast_base = forecast_base[forecast_base['SUB REGION'].astype(str).isin(region_filter)]
            if model_filter: forecast_base = forecast_base[forecast_base['MODELO OPPO'].isin(model_filter)]
            if channel_filter: forecast_base = forecast_base[forecast_base['Channel'].isin(channel_filter)]
            
            # Calculate Base (Current Average)
            weeks_count = forecast_base['WEEK'].astype(str).nunique()
            if weeks_count == 0: weeks_count = 1
            current_weekly_volume = forecast_base['SHOP SO'].sum() / weeks_count
            
            with c_set1:
                st.markdown("### 2. 基準校正")
                st.caption(f"目前數據平均週銷: {int(current_weekly_volume):,}")
                manual_base = st.number_input("手動調整基準週銷量 (可選)", value=int(current_weekly_volume), step=10)
                final_base_volume = manual_base if manual_base > 0 else 1

            # Editable Seasonality Table
            with c_set2:
                st.markdown("### 3. 季節係數 (Seasonality Index)")
                if 'seasonality_df' not in st.session_state:
                    default_season = pd.DataFrame({
                        'Month': ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
                        'Index': [1.3, 0.9, 0.9, 1.05, 1.7, 1.3, 1.25, 0.9, 1.0, 1.2, 2.0, 1.8]
                    })
                    st.session_state.seasonality_df = default_season
                
                edited_seasonality = st.data_editor(
                    st.session_state.seasonality_df,
                    column_config={
                        "Index": st.column_config.NumberColumn("Weight (1.0 = Avg)", min_value=0.1, max_value=5.0, step=0.1)
                    },
                    hide_index=True,
                    use_container_width=True
                )
                season_map = dict(zip(edited_seasonality['Month'], edited_seasonality['Index']))

            # Segment Distribution Logic (UPDATED V36.0)
            def get_forecast_tier(price):
                if price < 3000: return "Unknown"
                if price <= 4499: return t('seg_low')      # A LOW
                if price <= 5999: return t('seg_mid')      # A MEDIUM
                if price <= 7999: return t('seg_mid_high') # A HIGH
                return t('seg_high')                       # RENO + FIND

            forecast_base['ASP'] = forecast_base['MODELO OPPO'].map(price_map).fillna(0)
            forecast_base['Price_Segment'] = forecast_base['ASP'].apply(get_forecast_tier)
            forecast_base = forecast_base[forecast_base['Price_Segment'] != "Unknown"]
            
            # Calculate Share per Segment based on filtered history
            seg_share = forecast_base.groupby('Price_Segment')['SHOP SO'].sum().reset_index()
            total_vol = seg_share['SHOP SO'].sum()
            seg_share['Share'] = seg_share['SHOP SO'] / total_vol if total_vol > 0 else 0
            share_map = dict(zip(seg_share['Price_Segment'], seg_share['Share']))

            # Generate Forecast Data
            forecast_rows = []
            month_list = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
            
            # Simple 4-4-5 mapping approximation for visualization
            weeks_per_month = [4, 4, 5, 4, 4, 5, 4, 4, 5, 4, 4, 5] 
            
            current_week = 1
            for m_idx, month in enumerate(month_list):
                month_factor = season_map.get(month, 1.0)
                n_weeks = weeks_per_month[m_idx]
                
                # Monthly Total for this month
                monthly_total_base = final_base_volume * n_weeks * (1 + growth_2026) * month_factor
                
                for w in range(n_weeks):
                    for seg, share in share_map.items():
                        vol = (monthly_total_base / n_weeks) * share
                        forecast_rows.append({
                            'Month': month,
                            'Week_Num': current_week,
                            'Price_Segment': seg,
                            'Forecast_Sales': vol
                        })
                    current_week += 1

            df_forecast = pd.DataFrame(forecast_rows)
            
            st.divider()
            
            if not df_forecast.empty:
                c_chart1, c_chart2 = st.columns(2)
                
                with c_chart1:
                    st.markdown("#### 📅 月份視圖 (Monthly View)")
                    monthly_agg = df_forecast.groupby(['Month', 'Price_Segment'])['Forecast_Sales'].sum().reset_index()
                    # Sort by Month Order
                    monthly_agg['Month'] = pd.Categorical(monthly_agg['Month'], categories=month_list, ordered=True)
                    monthly_agg = monthly_agg.sort_values('Month')
                    
                    # Updated Order for V36.0
                    fig_month = px.bar(
                        monthly_agg, x='Month', y='Forecast_Sales', color='Price_Segment',
                        category_orders={"Price_Segment": [t('seg_low'), t('seg_mid'), t('seg_mid_high'), t('seg_high')]},
                        color_discrete_sequence=['#E0E0E0', '#999999', '#4D4D4D', '#000000']
                    )
                    st.plotly_chart(fig_month, use_container_width=True)
                
                with c_chart2:
                    st.markdown("#### 📈 週次視圖 (Weekly Trend)")
                    # Updated Order for V36.0
                    fig_week = px.area(
                        df_forecast, x='Week_Num', y='Forecast_Sales', color='Price_Segment',
                        category_orders={"Price_Segment": [t('seg_low'), t('seg_mid'), t('seg_mid_high'), t('seg_high')]},
                        color_discrete_sequence=['#E0E0E0', '#999999', '#4D4D4D', '#000000']
                    )
                    st.plotly_chart(fig_week, use_container_width=True)

                # Total Table
                total_2026 = df_forecast['Forecast_Sales'].sum()
                st.metric("🏆 2026 全年預測總量", f"{int(total_2026):,}")
                
                # Summary Table (V36.0)
                st.markdown("### 📊 總量細項 (Summary)")
                summary_table = df_forecast.groupby('Price_Segment')['Forecast_Sales'].sum().reset_index()
                summary_table['Monthly_Avg'] = summary_table['Forecast_Sales'] / 12
                # Sort manually
                sorter = {t('seg_low'): 1, t('seg_mid'): 2, t('seg_mid_high'): 3, t('seg_high'): 4}
                summary_table['Order'] = summary_table['Price_Segment'].map(sorter)
                summary_table = summary_table.sort_values('Order').drop('Order', axis=1)
                
                st.dataframe(
                    summary_table.style.format({'Forecast_Sales': '{:,.0f}', 'Monthly_Avg': '{:,.0f}'}), 
                    use_container_width=True
                )
            
        else:
            st.info("請先上傳檔案以產生預測 (Please upload files).")

    with tabs[9]: # NEW: TAB 9 (S-Class Stores Analysis with SHOP NUMBER Fix) (Unchanged logic)
        st.subheader(t('s_class_title'))
        
        if not df_calc.empty:
            # 1. Aggregate Sales by Store
            store_performance = df_calc.groupby(['SHOP NAME', 'SHOP NUMBER', 'SUB REGION', 'Channel']).agg({
                'Predicted_Sales': 'sum',
                'SHOP INV': 'sum',
                'Suggested_PO_Qty': 'sum'
            }).reset_index()
            
            # 2. Sort Descending
            store_performance = store_performance.sort_values('Predicted_Sales', ascending=False)
            
            # 3. Calculate Top 20%
            total_stores = len(store_performance)
            top_20_count = int(total_stores * 0.2)
            if top_20_count == 0: top_20_count = 1
            
            s_class = store_performance.head(top_20_count).copy()
            
            # 4. Add WoW if available
            if 'all_trends_granular' in st.session_state:
                trends = st.session_state.all_trends_granular
                # Calc last week vs prev week for each store
                latest_wk = trends['WEEK'].max()
                if latest_wk and str(latest_wk).isdigit():
                    prev_wk = str(int(latest_wk) - 1)
                    
                    s_trend = trends[trends['WEEK'].astype(str).isin([latest_wk, prev_wk])].groupby(['SHOP NUMBER', 'WEEK'])['SHOP SO'].sum().unstack(fill_value=0)
                    if latest_wk in s_trend.columns and prev_wk in s_trend.columns:
                        s_trend['WoW'] = ((s_trend[latest_wk] - s_trend[prev_wk]) / (s_trend[prev_wk] + 0.1) * 100)
                        s_class = pd.merge(s_class, s_trend[['WoW']], on='SHOP NUMBER', how='left')
            
            # 5. AI Diagnostics
            def get_s_insight(row):
                wos = row['SHOP INV'] / row['Predicted_Sales'] if row['Predicted_Sales'] > 0 else 99
                wow = row.get('WoW', 0)
                
                if wos < 3: return "🚨 嚴重缺貨 (Urgent OOS)"
                if wos < 5: return "⚠️ 庫存偏低 (Low Stock)"
                if wow < -20: return "📉 動能下滑 (Momentum Lost)"
                if wos > 12: return "🐢 庫存積壓 (Overstock)"
                return "✅ 健康 (Healthy)"

            s_class['Insight'] = s_class.apply(get_s_insight, axis=1)
            
            # 6. Display
            st.metric("S-Class Stores Count", f"{len(s_class)} / {total_stores}")
            
            def style_insight(val):
                if "🚨" in val: return "background-color: #FF5252; color: white"
                if "⚠️" in val: return "background-color: #FFD700; color: black"
                if "✅" in val: return "color: green"
                if "📉" in val: return "color: red"
                return ""

            # --- ADDED SHOP NUMBER AT THE BEGINNING ---
            cols_to_show = ['SHOP NUMBER', 'SHOP NAME', 'SUB REGION', 'Channel', 'Predicted_Sales', 'SHOP INV', 'Suggested_PO_Qty', 'WoW', 'Insight']
            if 'WoW' not in s_class.columns: cols_to_show.remove('WoW')
            
            st.dataframe(
                s_class[cols_to_show]
                .style.format({'Predicted_Sales': '{:.1f}', 'SHOP INV': '{:.0f}', 'Suggested_PO_Qty': '{:.0f}', 'WoW': '{:+.1f}%'})
                .applymap(style_insight, subset=['Insight']),
                use_container_width=True
            )
            
            # 7. S-Class Trend Chart
            if 'all_trends_granular' in st.session_state:
                s_ids = s_class['SHOP NUMBER'].tolist()
                s_history = st.session_state.all_trends_granular[st.session_state.all_trends_granular['SHOP NUMBER'].isin(s_ids)]
                s_history_agg = s_history.groupby('WEEK')['SHOP SO'].sum().reset_index()
                s_history_agg['WEEK_NUM'] = pd.to_numeric(s_history_agg['WEEK'])
                s_history_agg = s_history_agg.sort_values('WEEK_NUM')
                
                st.markdown("#### 📈 S 級門店總銷量趨勢 (S-Class Trend)")
                fig_s = px.line(s_history_agg, x='WEEK', y='SHOP SO', markers=True, title="Total Weekly Sales of Top 20% Stores")
                st.plotly_chart(fig_s, use_container_width=True)

        else:
            st.info("No data to calculate S-Class stores.")

    # --- NEW: TAB 10 (Independent CAPA Analysis) ---
    with tabs[10]:
        st.subheader(t('capa_title_main'))
        
        # --- 1. Seasonality Config Section ---
        st.markdown(t('capa_season_header'))
        if 'capa_season_df' not in st.session_state:
            # Default Factors as requested: Jan-Mar 0.9, Apr 1, May 1.2, Jun-Aug 1, Sep 0.9, Oct 1, Nov-Dec 1.3
            st.session_state.capa_season_df = pd.DataFrame({
                'Month': ['None', 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
                'Factor': [1.0, 0.9, 0.9, 0.9, 1.0, 1.2, 1.0, 1.0, 1.0, 0.9, 1.0, 1.3, 1.3]
            })
        
        c_season1, c_season2 = st.columns([1, 1])
        with c_season1:
            edited_capa_season = st.data_editor(
                st.session_state.capa_season_df[st.session_state.capa_season_df['Month'] != 'None'], # Hide 'None' from editing
                column_config={
                    "Factor": st.column_config.NumberColumn("Factor (e.g. 1.2)", min_value=0.1, max_value=5.0, step=0.1)
                },
                hide_index=True,
                use_container_width=True,
                key="capa_season_editor" # Key to avoid rerun issues
            )
            # Update non-None values back to state
            st.session_state.capa_season_df.loc[st.session_state.capa_season_df['Month'].isin(edited_capa_season['Month']), 'Factor'] = edited_capa_season['Factor'].values
        
        with c_season2:
            month_list = st.session_state.capa_season_df['Month'].tolist()
            # Default to None
            selected_capa_month = st.selectbox(t('capa_month_select'), month_list, index=0)
            
            applied_factor = st.session_state.capa_season_df.loc[st.session_state.capa_season_df['Month'] == selected_capa_month, 'Factor'].values[0]
            st.metric(t('capa_applied_factor'), f"{applied_factor}x")

        st.divider()

        # --- 2. CAPA Calculation Logic (Independent) ---
        # Load independent CAPA data directly
        df_capa_tool = pd.DataFrame(DEFAULT_CAPA_DATA)
        
        if not df_capa_tool.empty:
            # Calculation: Adjusted CAPA = Original CAPA * Factor
            # Meaning: In peak season (factor > 1), effective capacity might need to be higher, or we simulate higher stock load?
            # User requirement: "Use preset CAPA to multiply/divide... then show variation below"
            # Interpretation: 
            # If factor = 1.2 (May), Adjusted CAPA = 100 * 1.2 = 120.
            # This shows "Projected Capacity Need" or "Projected Load"?
            # Let's assume Adjusted CAPA = CAPA * Factor.
            
            df_capa_tool['Adjusted_CAPA'] = df_capa_tool['CAPA_QTY'] * applied_factor
            
            # Only show Delta if Factor != 1
            if applied_factor != 1.0:
                df_capa_tool['Delta'] = df_capa_tool['Adjusted_CAPA'] - df_capa_tool['CAPA_QTY']
            else:
                df_capa_tool['Delta'] = 0

            # Display Metrics
            total_orig_capa = df_capa_tool['CAPA_QTY'].sum()
            total_adj_capa = df_capa_tool['Adjusted_CAPA'].sum()
            
            m1, m2, m3 = st.columns(3)
            m1.metric("Total Original CAPA", f"{int(total_orig_capa):,}")
            m2.metric(f"Total Adjusted CAPA ({selected_capa_month})", f"{int(total_adj_capa):,}", delta=f"{int(total_adj_capa - total_orig_capa):,}")
            
            st.markdown("### 📋 門店 CAPA 詳細數據 (Store Details)")
            
            cols_to_show = ['SHOP NUMBER', 'SUB REGION', 'SHOP NAME', 'CAPA_QTY', 'Adjusted_CAPA', 'Delta']
            
            st.dataframe(
                df_capa_tool[cols_to_show]
                .sort_values('Adjusted_CAPA', ascending=False)
                .style.format({
                    'CAPA_QTY': '{:,.0f}',
                    'Adjusted_CAPA': '{:,.0f}',
                    'Delta': '{:+,.0f}'
                })
                .background_gradient(subset=['Adjusted_CAPA'], cmap='Greens'),
                use_container_width=True,
                hide_index=True
            )
            
            # Visualization
            st.markdown("#### 📊 CAPA 分佈預覽 (Distribution Preview)")
            fig_capa = px.bar(df_capa_tool.head(20), x='SHOP NAME', y=['CAPA_QTY', 'Adjusted_CAPA'], 
                              barmode='group', title=f"Top 20 Stores: Original vs Adjusted CAPA ({selected_capa_month})")
            st.plotly_chart(fig_capa, use_container_width=True)

        else:
            st.info("No CAPA data available.")

else:
    st.info(t('upload_prompt'))