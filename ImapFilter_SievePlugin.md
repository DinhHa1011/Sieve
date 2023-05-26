## IMAP FILTER=SIEVE Plugin
- Thông thường, Sieve filter có thể được áp dụng khi gửi mail ban đầu hoặc được kích hoạt bởi các sự kiện nhất định trong giao thức IMAP (IMAPSIEVEC;RFC 6785)
- Người dùng có thể config Sieve script nào sẽ chạy trong các trường hợp này, nhưng nó không thể kích hoạt việc thực thi các Sieve scripts theo cách thủ công
- Tuy nhiên, điều này có thể rất hữu ích, ví dụ: để kiểm tra các Sieve rules mới và lọc lại các message đã bị xử lý sai bởi version trước đó của Sieve script có liên quan
- Pigeonhole cung cấp `imap_filter_sieve` plugin, nơi cung cấp IMAP extension gọi là `FILTER=SIEVE`
- Điều này thêm một dòng lệnh `FILTER` cho phép áp dụng mail filter (Sieve script) trên cài đặt của message để match với tiêu chí tìm kiếm hình ảnh được chỉ định
- Bản nháp mới nhất của thông số kỹ thuật cho khả năng IMAP này có sẵn tại đây. Plugin này là thử nghiệm và thông số kỹ thuật có thể thay đổi, sử dụng thông số kỹ thuật có trong bản phát hành hiện tại của bạn dể có được thông số kỹ thuật phù hợp cho bản phát hành của bạn
- Plugin này có sẵn cho Pigeonhole v0.4.24 trở lên (có sẵn cho Doveoct v2.2.36) và v0.5.2 trở lên (có sẵn cho Dovecot v2.3.2)
- Các plugin được bao gồm trong gói Pigeonhole và do đó được biên dịch và cài đặt hoàn toàn với chính Pigeonhole
### Setting
- See https://doc.dovecot.org/settings/plugin/imap-filter-sieve-plugin/#imap-filter-sieve
### Configuration
- IMAP FILTER Sieve plugin được kích hoạt bằng cách thêm nó vào cài đặt `mail_plugins` cho imap protocol:
```
protocol imap {
  mail_plugins = $mail_plugins imap_filter_sieve
}
```
- Lưu ý rằng enable plugin này cho phép người dùng chỉ định nội dung Sieve script như một tham số cho dòng lệnh `FILTER`, không chỉ run các script được lưu trữ hiện có
- Plugin này sử dụng cài đặt config bình thường được sử dụng bởi LDA Sieve plugin lúc delivery
- `sieve_before` và `sieve_after` script hiện đang bị plugin này bỏ qua
## IMAPSIEVE
- Như được định nghĩa trong thông số kỹ thuật cơ sở RFC 5228, ngôn ngữ Sieve chỉ được sử dụng trong quá trình phân phối
- Tuy nhiên, về nguyên tắc, nó có thể được sử dụng tại bất kỳ thời điểm nào trong quá trình xử lý email
- RFC 6785 xác định việc sử dụng Sieve filter trong IMAP => hoạt động khi mail được tạo hoặc thuộc tính của chúng bị thay đổi, tính năng này mở rộng cả Sieve và IMAP
- Do đó, Pigeonhole cung cấp cả IMAP plugin và Sieve plugin
- `sieve_imapsieve` triển khai `imapsieve` extension cho ngôn ngữ Sieve filter, thêm chức năng sử dụng Sieve scripts từ bên trong IMAP
- `imap_sieve` plugin cho IMAP bổ sung khả năng IMAPSIEVE cho dịch vụ imap
- khả năng `IMAPSIEVE` cơ bản cho phép đính kèm Sieve script vào mailbox cho bất kỳ mailbox nào bằng cách đặt mục nhập siêu dữ liệu ảnh đặc biệt. Bằng cách này, người dùng có thể định cấu hình các Sieve script chạy cho các sự kiện IMAP trong mailbox của họ
- Ngoài tiêu chuẩn, việc triển khai Pigeon cũng bổ sung khả năng cho admin định cấu hình các Sieve script bên ngoài sự kiểm soát của user, được chạy trước hoặc sau script của người dùng nếu có
- Plugin này có sẵn cho Pigeonhole v0.4.14 trở lên (có sẵn cho Dovecot v2.2.24) các plugin được bao gồm trong gói Pieonhole và do đó được biên dịch và cài đặt ngầm với chính Pieonhole
### My Config
- Ví dụ dưới đây config khi nào người dùng move một message vào folder report_spam, một copy của message được đặt vào folder report_ham hoặc report_spam của spam@example.com maibox tương ứng
- Khi một message located trong folder `report_spam` reply hoặc forward, một bản copy của message được đặt vào folder `report_spam_reply`

`vim /etc/dovecot/dovecot.conf`
```
# Store METADATA information within user's HOME directory
mail_attribute_dict = file:%Lh/dovecot-attributes

protocol imap {
    ...
    mail_plugins = ... imap_sieve
}

plugin {
    sieve_plugins = sieve_imapsieve sieve_extprograms
    imapsieve_url = sieve://127.0.0.1:4190

    # From elsewhere to Junk folder
    imapsieve_mailbox1_name = Junk
    imapsieve_mailbox1_causes = COPY APPEND
    imapsieve_mailbox1_before = file:/var/vmail/sieve/report_spam.sieve

    # From Junk folder to elsewhere
    imapsieve_mailbox2_name = *
    imapsieve_mailbox2_from = Junk
    imapsieve_mailbox2_causes = COPY
    imapsieve_mailbox2_before = file:/var/vmail/sieve/report_ham.sieve

    sieve_pipe_bin_dir = /etc/dovecot/sieve/pipe

    sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.environment

}
```
Tạo thư mục, file, script
```
mkdir -p /etc/dovecot/sieve/pipe
mkdir -p /var/vmail/imapsieve_copy
chown vmail:vmail /var/vmail/imapsieve_copy
chmod 0700 /var/vmail/imapsieve_copy
```
`vim /var/vmail/sieve/report_spam.sieve`
```
require ["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "imap4flags"];

if environment :is "imap.cause" "COPY" {
    pipe :copy "dovecot-lda" [ "-d", "ah@dinhha.online", "-m", "report_spam" ];
}

# Catch replied or forwarded spam
elsif anyof (allof (hasflag "\\Answered",
                    environment :contains "imap.changedflags" "\\Answered"),
             allof (hasflag "$Forwarded",
                    environment :contains "imap.changedflags" "$Forwarded")) {
    pipe :copy "dovecot-lda" [ "-d", "ah@dinhha.online", "-m", "report_spam_reply" ];
}

```
`vim /var/vmail/sieve/report_ham.sieve`
```
require ["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"];

if environment :matches "imap.mailbox" "*" {
  set "mailbox" "${1}";
}

if string "${mailbox}" [ "Trash", "train_ham", "train_prob", "train_spam" ] {
  stop;
}

pipe :copy "dovecot-lda" [ "-d", "ah@dinhha.online", "-m", "report_ham" ];
```
`vim /etc/dovecot/sieve/pipe/imapsieve_copy`
```
#!/usr/bin/env bash
# Author: Zhang Huangbin <zhb@iredmail.org>
# Purpose: Read full email message from stdin, and save to a local file.

# Usage: bash imapsieve_copy <email> <spam|ham> <output_base_dir>

export USER="$1"
export MSG_TYPE="$2"

export OUTPUT_BASE_DIR="/var/vmail/imapsieve_copy"
export OUTPUT_DIR="${OUTPUT_BASE_DIR}/${MSG_TYPE}"
export FILE="${OUTPUT_DIR}/${USER}-$(date +%Y%m%d%H%M%S)-${RANDOM}${RANDOM}.eml"

export OWNER="vmail"
export GROUP="vmail"

for dir in "${OUTPUT_BASE_DIR}" "${OUTPUT_DIR}"; do
    if [[ ! -d ${dir} ]]; then
        mkdir -p ${dir}
        chown ${OWNER}:${GROUP} ${dir}
        chmod 0700 ${dir}
    fi
done

cat > ${FILE} < /dev/stdin

# Logging
#export LOG='logger -p local5.info -t imapsieve_copy'
#[[ $? == 0 ]] && ${LOG} "Copied one ${MSG_TYPE} email reported by ${USER}: ${FILE}"
```
- Set correct file owner and permissions
```
chown vmail:vmail /var/vmail/sieve/report_spam.sieve \
    /var/vmail/sieve/report_ham.sieve \
    /etc/dovecot/sieve/pipe/imapsieve_copy

chmod 0700 /var/vmail/sieve/report_spam.sieve \
    /var/vmail/sieve/report_ham.sieve \
    /etc/dovecot/sieve/pipe/imapsieve_copy
```
- Setup cron job to scan and learn spam/ham messages
`vim /etc/dovecot/sieve/scan_reported_mails.sh`
```
#!/usr/bin/env bash
# Author: Zhang Huangbin <zhb@iredmail.org>
# Purpose: Copy spam/ham to another directory and call sa-learn to learn.

# Paths to find program.
export PATH="/bin:/usr/bin:/usr/local/bin:$PATH"

export OWNER="vmail"
export GROUP="vmail"

# The Amavisd daemon user.
# Note: on OpenBSD, it's "_vscan". On FreeBSD, it's "vscan".
export AMAVISD_USER='amavis'
export AMAVISD_USER_HOMEDIR="$(eval echo ~${AMAVISD_USER})"

# Kernel name, in upper cases.
export KERNEL_NAME="$(uname -s | tr '[a-z]' '[A-Z]')"

# A temporary lock file. should be removed after successfully examed messages.
export LOCK_FILE='/tmp/scan_reported_mails.lock'

# Logging to syslog with 'logger' command.
export LOG='logger -p local5.info -t scan_reported_mails'

# `sa-learn` command, with optional arguments.
export SA_LEARN="sa-learn -u ${AMAVISD_USER} --dbpath ${AMAVISD_USER_HOMEDIR}/.spamassassin"

# Spool directory.
# Must be owned by vmail:vmail.
export SPOOL_DIR='/var/vmail/imapsieve_copy'

# Directories which store spam and ham emails.
# These 2 should be created while setup Dovecot antispam plugin.
export SPOOL_SPAM_DIR="${SPOOL_DIR}/spam"
export SPOOL_HAM_DIR="${SPOOL_DIR}/ham"

# Directory used to store emails we're going to process.
# We will copy new spam/ham messages to these directories, scan them, then
# remove them.
export SPOOL_LEARN_SPAM_DIR="${SPOOL_DIR}/processing/spam"
export SPOOL_LEARN_HAM_DIR="${SPOOL_DIR}/processing/ham"

if [ -e ${LOCK_FILE} ]; then
    find $(dirname ${LOCK_FILE}) -maxdepth 1 -ctime 1 "$(basename ${LOCK_FILE})" >/dev/null 2>&1
    if [ X"$?" == X'0' ]; then
        rm -f ${LOCK_FILE} >/dev/null 2>&1
    else
        ${LOG} "Lock file exists (${LOCK_FILE}), abort."
        exit
    fi
fi

for dir in "${SPOOL_DIR}" "${SPOOL_LEARN_SPAM_DIR}" "${SPOOL_LEARN_HAM_DIR}"; do
    if [[ ! -d ${dir} ]]; then
        mkdir -p ${dir}
    fi

    chown ${OWNER}:${GROUP} ${dir}
    chmod 0700 ${dir}
done

# If there're a lot files, direct `mv` command may fail with error like
# `argument list too long`, so we need `find` in this case.
if [[ X"${KERNEL_NAME}" == X'OPENBSD' ]] || [[ X"${KERNEL_NAME}" == X'FREEBSD' ]]; then
    [[ -d ${SPOOL_SPAM_DIR} ]] && find ${SPOOL_SPAM_DIR} -name '*.eml' -exec mv {} ${SPOOL_LEARN_SPAM_DIR}/ \;
    [[ -d ${SPOOL_HAM_DIR} ]]  && find ${SPOOL_HAM_DIR}  -name '*.eml' -exec mv {} ${SPOOL_LEARN_HAM_DIR}/  \;
else
    [[ -d ${SPOOL_SPAM_DIR} ]] && find ${SPOOL_SPAM_DIR} -name '*.eml' -exec mv -t ${SPOOL_LEARN_SPAM_DIR}/ {} +
    [[ -d ${SPOOL_HAM_DIR} ]]  && find ${SPOOL_HAM_DIR}  -name '*.eml' -exec mv -t ${SPOOL_LEARN_HAM_DIR}/  {} +
fi

# Try to delete empty directory, if failed, that means we have some messages to
# scan.
rmdir ${SPOOL_LEARN_SPAM_DIR} &>/dev/null
if [[ X"$?" != X'0' ]]; then
    output="$(${SA_LEARN} --spam ${SPOOL_LEARN_SPAM_DIR})"
    rm -rf ${SPOOL_LEARN_SPAM_DIR} &>/dev/null
    ${LOG} '[SPAM]' ${output}
fi

rmdir ${SPOOL_LEARN_HAM_DIR} &>/dev/null
if [[ X"$?" != X'0' ]]; then
    output="$(${SA_LEARN} --ham ${SPOOL_LEARN_HAM_DIR})"
    rm -rf ${SPOOL_LEARN_HAM_DIR} &>/dev/null
    ${LOG} '[CLEAN]' ${output}
fi

rm -f ${LOCK_FILE} &>/dev/null
```
- Run command `crontab -e -u root` để setup cron job cho root user, scan email mỗi 10 phút
```
# iRedMail: Scan reported mails.
*/10   *   *   *   *   /bin/bash /etc/dovecot/sieve/scan_reported_mails.sh
```
