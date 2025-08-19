FROM python:3.10

# 1. Cài đặt gói hệ thống đầy đủ
RUN apt-get update && apt-get install -y \
    wget unzip curl gnupg ca-certificates jq \
    build-essential libffi-dev libssl-dev libpq-dev \
    libxml2-dev libxslt1-dev libaio1 alien \
    fonts-liberation libappindicator3-1 libasound2 \
    libatk-bridge2.0-0 libatk1.0-0 libcups2 \
    libdbus-1-3 libgdk-pixbuf2.0-0 libnspr4 libnss3 \
    libx11-xcb1 libxcomposite1 libxdamage1 libxrandr2 \
    libgbm1 libgtk-3-0 libxss1 libxshmfence1 nano \
    && rm -rf /var/lib/apt/lists/*

# 2. Cài Google Chrome
RUN wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && apt-get install -y google-chrome-stable && \
    rm -rf /var/lib/apt/lists/*

# 3. Cài ChromeDriver tương ứng
RUN CHROME_VERSION=$(google-chrome-stable --version | grep -oP "\d+\.\d+\.\d+\.\d+") && \
    DRIVER_URL=$(curl -s "https://googlechromelabs.github.io/chrome-for-testing/known-good-versions-with-downloads.json" | \
    jq -r --arg ver "$CHROME_VERSION" '.versions[] | select(.version==$ver) | .downloads.chromedriver[] | select(.platform=="linux64") | .url') && \
    mkdir -p /tmp/chromedriver && cd /tmp/chromedriver && \
    wget "$DRIVER_URL" -O chromedriver.zip && unzip chromedriver.zip && \
    mv chromedriver-linux64/chromedriver /usr/local/bin/chromedriver && chmod +x /usr/local/bin/chromedriver && \
    rm -rf /tmp/chromedriver

# 4. Cài Oracle Instant Client + SQL*Plus
WORKDIR /opt/oracle
COPY instantclient-basiclite-linux.x64-21.19.0.0.0dbru.zip .
COPY instantclient-sqlplus-linux.x64-21.19.0.0.0dbru.zip .

RUN unzip -o instantclient-basiclite-linux.x64-21.19.0.0.0dbru.zip && \
    unzip -o instantclient-sqlplus-linux.x64-21.19.0.0.0dbru.zip && \
    ln -s /opt/oracle/instantclient_21_19 /opt/oracle/instantclient && \
    echo /opt/oracle/instantclient > /etc/ld.so.conf.d/oracle-instantclient.conf && \
    ldconfig

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient
ENV PATH="$PATH:/opt/oracle/instantclient"

# 5. Cài Airflow ổn định (SQLAlchemy < 2.0) + driver Postgres
RUN pip install --no-cache-dir "apache-airflow==2.10.3" psycopg2-binary

# 6. Tạo virtualenv riêng cho Oracle (SQLAlchemy ≥ 2.0 + oracledb)
RUN python -m venv /opt/oracle_venv && \
    /opt/oracle_venv/bin/pip install --no-cache-dir --upgrade pip && \
    /opt/oracle_venv/bin/pip install --no-cache-dir \
        "SQLAlchemy>=2.0.0" \
        oracledb \
        cx_Oracle \
        selenium \
        selenium-stealth \
        beautifulsoup4 \
        pandas \
        numpy \
        openpyxl \
        XlsxWriter \
        python-dateutil \
        pytz \
        holidays \
        trio \
        trio-websocket \
        PySocks \
        sortedcontainers \
        sniffio \
        websocket-client \
        typing_extensions \
        certifi \
        urllib3 \
        idna \
        attrs \
        h11 \
        outcome \
        exceptiongroup \
        async-generator \
        et-xmlfile \
        six \
        pycparser \
        tzdata \
        lxml

# 7. Cài thêm thư viện Python chung
RUN pip install --no-cache-dir \
    selenium \
    selenium-stealth \
    beautifulsoup4 \
    pandas \
    numpy \
    openpyxl \
    XlsxWriter \
    python-dateutil \
    pytz \
    holidays \
    trio \
    trio-websocket \
    PySocks \
    sortedcontainers \
    sniffio \
    websocket-client \
    typing_extensions \
    certifi \
    urllib3 \
    idna \
    attrs \
    h11 \
    outcome \
    exceptiongroup \
    async-generator \
    et-xmlfile \
    six \
    pycparser \
    tzdata \
    lxml

# 8. Tạo thư mục Airflow
RUN mkdir -p /home/airflow/dags /home/airflow/logs /home/airflow/plugins
WORKDIR /home/airflow

CMD ["/bin/bash"]