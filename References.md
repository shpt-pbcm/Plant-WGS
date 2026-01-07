# Plant-WGS

## Paper
### Can we use it? On the utility of de novo and reference-based assembly of Nanopore data for plant plastome sequencing
[Link to paper](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0226234)

#### Pipeline

1. **Xử lý dữ liệu đọc Nanopore**
   - Dữ liệu Nanopore được **base-calling** bằng Albacore v2.3.4, chỉ giữ các read đạt Q-score.
   - Các read đạt chuẩn được **demultiplex**, loại adapter và barcode bằng **Porechop**, đồng thời loại bỏ read nghi ngờ chimera.
   - **Primer** được cắt bằng **BBDuk** với tham số tối ưu cho dữ liệu Nanopore.
   - Sau đó, read được lọc theo chiều dài và chất lượng bằng **NanoFilt**. Các read sau xử lý được gọi là **processed reads**.
   - Đối với **lắp ráp de novo**, dữ liệu tiếp tục được lọc nhiễm (chloroplast **Artemisia frigida**), tách chimera bằng **Pacasus**, loại bỏ read quá ngắn, tạo ra tập **fully processed reads**.

2. **Lắp ráp dựa trên tham chiếu (Reference-based assembly – Nanopore)**
   - Các read được chia nhỏ và ánh xạ lên **genome tham chiếu** bằng **BBMap** (`mapPacBio.sh`).
   - Gọi biến thể được thực hiện bằng **callvariants2.sh**, có **realignment** để cải thiện độ chính xác quanh vùng **indel**.
   - Trình tự **consensus** được tạo từ file **VCF**.

3. **Lắp ráp de novo bằng Nanopore**
   - Tập **fully processed reads** được lắp ráp bằng **Canu v1.8**, với tham số tối ưu cho genome chloroplast và dữ liệu độ phủ cao.
   - Các **contig kém chất lượng** bị loại bỏ. 
   - **Contig** được kiểm tra nhiễm/chimera bằng **BLAST**, chỉnh sửa nếu cần, sau đó đánh bóng bằng **Nanopolish** dựa trên file **fast5** gốc.
   - Các contig cuối được **sắp xếp**, **nối thủ công** và bổ sung vùng **IRA** để hoàn chỉnh genome.

4. **Lắp ráp de novo lai (Hybrid assembly – Nanopore + Illumina)**
   - Hai chiến lược được áp dụng:
     - Lắp ráp trực tiếp bằng **Unicycler** với dữ liệu **Nanopore + Illumina**.
     - Sửa lỗi read **Nanopore** bằng **Illumina** thông qua **Nanocorr**, sau đó lắp ráp lại bằng **Canu**. Trình tự cuối được xử lý tương tự lắp ráp de novo Nanopore, nhưng **không đánh bóng thêm** để tránh suy giảm chất lượng.

5. **Chú giải genome chloroplast**
   - Các genome hoàn chỉnh được **chú giải** bằng **GeSeq**, sử dụng genome tham chiếu họ hàng gần.
   - Kết quả được kiểm tra chéo bằng **HMMER** và **ARAGORN**, chỉnh sửa thủ công các vùng chưa chính xác.
   - **Bản đồ genome** được vẽ bằng **OGDRAW** và chuẩn bị nộp **GenBank** bằng **GB2sequin**.

6. **So sánh kết quả lắp ráp**
   - Các genome thu được từ các pipeline khác nhau được **so sánh** với **Illumina de novo assembly** (không tính IRA).
   - Trình tự được căn chỉnh bằng **MAFFT**, sau đó thống kê sai khác (SNP, insertion, deletion) bằng script **alignment_info3.sh**.
   - **Độ dài** và **GC%** được xác định bằng **BioEdit**.

7. **Phân tích sai khác trình tự và xác định marker phân tử**
   - Read **Illumina** của **L. virgatum** được ánh xạ lên **genome L. vulgare** để xác định vùng biến thiên.
   - Biến thể được gọi và phân tích mật độ bằng **VCFtools** theo cửa sổ 1,000 bp, từ đó xác định các vùng có mức đa hình cao, tiềm năng làm **marker phân tử**.
