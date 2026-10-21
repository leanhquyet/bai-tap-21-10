4.7
public class Student
{
    public string StudentId { get; set; }
    public string FullName { get; set; }
    public int Age { get; set; }
    public string Gender { get; set; }
}
using System;
using System.Drawing;
using System.Windows.Forms;

namespace MdiStudentApp
{
    public partial class FormMain : Form
    {
        private Timer statusTimer;

        public FormMain()
        {
            InitializeComponent();
            SetupMdiContainer();
            SetupStatusStrip();
        }

        private void SetupMdiContainer()
        {
            this.IsMdiContainer = true;
            this.Text = "Hệ thống Quản lý Đào tạo - MDI Application";
            this.WindowState = FormWindowState.Maximized;
        }

        private void SetupStatusStrip()
        {
            lblUser.Text = "Người dùng: Admin (Lê Anh Quyết)";
            lblStatus.Text = "Trạng thái: Sẵn sàng";

            statusTimer = new Timer();
            statusTimer.Interval = 1000;
            statusTimer.Tick += (s, e) => {
                lblTime.Text = DateTime.Now.ToString("dd/MM/yyyy HH:mm:ss");
            };
            statusTimer.Start();
        }

        private void OpenChildForm<T>() where T : Form, new()
        {
            foreach (Form child in this.MdiChildren)
            {
                if (child is T)
                {
                    child.Activate();
                    if (child.WindowState == FormWindowState.Minimized)
                    {
                        child.WindowState = FormWindowState.Normal;
                    }
                    return;
                }
            }

            T newChild = new T();
            newChild.MdiParent = this;
            newChild.Show();
        }

        private void mnuQuanLySinhVien_Click(object sender, EventArgs e)
        {
            OpenChildForm<FormStudentAsync>();
        }

        private void mnuLogout_Click(object sender, EventArgs e)
        {
            MessageBox.Show("Đã đăng xuất hệ thống!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }

        private void mnuExit_Click(object sender, EventArgs e)
        {
            Application.Exit();
        }

        private void cascadeToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.LayoutMdi(MdiLayout.Cascade);
        }

        private void tileHorizontalToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.LayoutMdi(MdiLayout.TileHorizontal);
        }

        private void tileVerticalToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.LayoutMdi(MdiLayout.TileVertical);
        }

        public void UpdateStatusMessage(string message)
        {
            lblStatus.Text = $"Trạng thái: {message}";
        }
    }
}
4.8
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace MdiStudentApp
{
    public partial class FormStudentAsync : Form
    {
        private BindingList<Student> studentList;
        private BindingSource bindingSource;

        public FormStudentAsync()
        {
            InitializeComponent();
            InitializeData();
            SetupContextMenu();
        }

        private void InitializeData()
        {
            studentList = new BindingList<Student>();
            bindingSource = new BindingSource();
            bindingSource.DataSource = studentList;
            dgvStudents.DataSource = bindingSource;
        }

        private void SetupContextMenu()
        {
            ContextMenuStrip contextMenu = new ContextMenuStrip();
            
            var itemRefresh = new ToolStripMenuItem("Làm mới dữ liệu");
            itemRefresh.Click += (s, e) => btnLoadData_Click(s, e);
            
            var itemDelete = new ToolStripMenuItem("Xóa dòng đang chọn");
            itemDelete.Click += (s, e) => {
                if (bindingSource.Current is Student st)
                {
                    studentList.Remove(st);
                    UpdateParentStatus($"Đã xóa sinh viên: {st.FullName}");
                }
            };

            var itemDetail = new ToolStripMenuItem("Xem chi tiết");
            itemDetail.Click += (s, e) => {
                if (bindingSource.Current is Student st)
                {
                    MessageBox.Show($"Mã: {st.StudentId}\nHọ tên: {st.FullName}\nTuổi: {st.Age}", "Chi tiết sinh viên", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
            };

            contextMenu.Items.AddRange(new ToolStripItem[] { itemRefresh, itemDelete, itemDetail });
            dgvStudents.ContextMenuStrip = contextMenu;
        }

        private async void btnLoadData_Click(object sender, EventArgs e)
        {
            btnLoadData.Enabled = false;
            progressBar1.Value = 0;
            lblPercent.Text = "0%";
            UpdateParentStatus("Đang tiến hành nạp dữ liệu sinh viên...");

            try
            {
                var progress = new Progress<int>(percent => {
                    progressBar1.Value = percent;
                    lblPercent.Text = $"{percent}%";
                    UpdateParentStatus($"Đang nạp dữ liệu... {percent}%");
                });

                List<Student> loadedStudents = await Task.Run(() => LoadDataFromDatabaseSimulation(progress));

                studentList.Clear();
                foreach (var st in loadedStudents)
                {
                    studentList.Add(st);
                }

                UpdateParentStatus("Nạp dữ liệu sinh viên thành công!");
                MessageBox.Show("Nạp thành công danh sách sinh viên!", "Thông báo", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Lỗi khi nạp dữ liệu: {ex.Message}", "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
                UpdateParentStatus("Nạp dữ liệu thất bại.");
            }
            finally
            {
                btnLoadData.Enabled = true;
            }
        }

        private List<Student> LoadDataFromDatabaseSimulation(IProgress<int> progress)
        {
            var list = new List<Student>();
            int totalRows = 100;

            for (int i = 1; i <= totalRows; i++)
            {
                System.Threading.Thread.Sleep(30); 

                list.Add(new Student 
                { 
                    StudentId = $"SV{i:D3}", 
                    FullName = $"Sinh viên số {i}", 
                    Age = 18 + (i % 5), 
                    Gender = (i % 2 == 0) ? "Nam" : "Nữ" 
                });

                int percent = (i * 100) / totalRows;
                progress?.Report(percent);
            }

            return list;
        }

        private void UpdateParentStatus(string message)
        {
            if (this.MdiParent is FormMain mainForm)
            {
                mainForm.UpdateStatusMessage(message);
            }
        }
    }
}
