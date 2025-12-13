<template>
  <div class="admin-panel">
    <div class="admin-header">
      <h1>👨‍💼 Admin Panel</h1>
      <p class="subtitle">Manage courses, students, and instructors</p>
    </div>

    <!-- Tab Navigation -->
    <div class="tab-nav">
      <button 
        @click="activeTab = 'courses'" 
        :class="['tab-btn', { active: activeTab === 'courses' }]"
      >
        📚 Courses
      </button>
      <button 
        @click="activeTab = 'students'" 
        :class="['tab-btn', { active: activeTab === 'students' }]"
      >
        👨‍🎓 Students
      </button>
      <button 
        @click="activeTab = 'instructors'" 
        :class="['tab-btn', { active: activeTab === 'instructors' }]"
      >
        👨‍🏫 Instructors
      </button>
    </div>

    <!-- Courses Tab -->
    <div v-if="activeTab === 'courses'" class="tab-content">
      <div class="content-header">
        <h2>Course Management</h2>
        <button @click="openCourseModal()" class="btn-create">
          + Create New Course
        </button>
      </div>

      <!-- Courses List -->
      <div v-if="loading" class="loading">Loading...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      <div v-else class="data-table">
        <table>
          <thead>
            <tr>
              <th>Code</th>
              <th>Name</th>
              <th>Credits</th>
              <th>Department</th>
              <th>Type</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="course in courses" :key="course.id">
              <td><strong>{{ course.course_code }}</strong></td>
              <td>{{ course.course_name }}</td>
              <td>{{ course.credits }}</td>
              <td>{{ course.dept_name }}</td>
              <td><span class="type-badge" :class="course.course_type">{{ formatType(course.course_type) }}</span></td>
              <td class="actions">
                <button @click="openCourseModal(course)" class="btn-edit">✏️ Edit</button>
                <button @click="deleteCourse(course)" class="btn-delete">🗑️ Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Students Tab -->
    <div v-if="activeTab === 'students'" class="tab-content">
      <div class="content-header">
        <h2>Student Management</h2>
      </div>

      <div v-if="loading" class="loading">Loading...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      <div v-else class="data-table">
        <table>
          <thead>
            <tr>
              <th>ID</th>
              <th>Name</th>
              <th>Email</th>
              <th>Major</th>
              <th>Year</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="student in students" :key="student.id">
              <td><strong>{{ student.student_id }}</strong></td>
              <td>{{ student.name }}</td>
              <td>{{ student.email }}</td>
              <td>{{ student.major }}</td>
              <td>{{ student.year_of_study }}</td>
              <td class="actions">
                <button @click="openStudentModal(student)" class="btn-edit">✏️ Edit</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Instructors Tab -->
    <div v-if="activeTab === 'instructors'" class="tab-content">
      <div class="content-header">
        <h2>Instructor Management</h2>
      </div>

      <div v-if="loading" class="loading">Loading...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      <div v-else class="data-table">
        <table>
          <thead>
            <tr>
              <th>ID</th>
              <th>Name</th>
              <th>Email</th>
              <th>Department</th>
              <th>Title</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="instructor in instructors" :key="instructor.id">
              <td><strong>{{ instructor.instructor_id }}</strong></td>
              <td>{{ instructor.name }}</td>
              <td>{{ instructor.email }}</td>
              <td>{{ instructor.dept_name }}</td>
              <td>{{ instructor.title }}</td>
              <td class="actions">
                <button @click="openInstructorModal(instructor)" class="btn-edit">✏️ Edit</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Course Modal -->
    <div v-if="showCourseModal" class="modal-overlay" @click="closeCourseModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>{{ editingCourse ? 'Edit Course' : 'Create New Course' }}</h2>
          <button @click="closeCourseModal" class="btn-close">×</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="saveCourse">
            <div class="form-group">
              <label>Course Code *</label>
              <input v-model="courseForm.course_code" :disabled="editingCourse" required />
            </div>
            <div class="form-group">
              <label>Course Name *</label>
              <input v-model="courseForm.course_name" required />
            </div>
            <div class="form-group">
              <label>Description</label>
              <textarea v-model="courseForm.description" rows="3"></textarea>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label>Credits *</label>
                <input v-model.number="courseForm.credits" type="number" min="1" max="10" required />
              </div>
              <div class="form-group">
                <label>Department *</label>
                <select v-model.number="courseForm.dept_id" required>
                  <option value="">Select...</option>
                  <option v-for="dept in departments" :key="dept.id" :value="dept.id">
                    {{ dept.dept_name }}
                  </option>
                </select>
              </div>
            </div>
            <div class="form-group">
              <label>Course Type *</label>
              <select v-model="courseForm.course_type" required>
                <option value="">Select...</option>
                <option value="general_required">General Required</option>
                <option value="major_required">Major Required</option>
                <option value="major_elective">Major Elective</option>
                <option value="university_elective">University Elective</option>
                <option value="practical">Practical</option>
              </select>
            </div>
            <div class="modal-actions">
              <button type="button" @click="closeCourseModal" class="btn-cancel">Cancel</button>
              <button type="submit" class="btn-save">{{ editingCourse ? 'Save Changes' : 'Create Course' }}</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Student Modal -->
    <div v-if="showStudentModal" class="modal-overlay" @click="closeStudentModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>Edit Student</h2>
          <button @click="closeStudentModal" class="btn-close">×</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="saveStudent">
            <div class="form-group">
              <label>Student ID</label>
              <input v-model="studentForm.student_id" disabled />
            </div>
            <div class="form-group">
              <label>Name *</label>
              <input v-model="studentForm.name" required />
            </div>
            <div class="form-group">
              <label>Email *</label>
              <input v-model="studentForm.email" type="email" required />
            </div>
            <div class="form-row">
              <div class="form-group">
                <label>Major</label>
                <input v-model="studentForm.major" />
              </div>
              <div class="form-group">
                <label>College</label>
                <input v-model="studentForm.college" />
              </div>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label>Year of Study</label>
                <select v-model.number="studentForm.year_of_study">
                  <option value="1">1st Year</option>
                  <option value="2">2nd Year</option>
                  <option value="3">3rd Year</option>
                  <option value="4">4th Year</option>
                  <option value="5">5th Year</option>
                </select>
              </div>
              <div class="form-group">
                <label>Gender</label>
                <select v-model="studentForm.gender">
                  <option value="">Select...</option>
                  <option value="Male">Male</option>
                  <option value="Female">Female</option>
                  <option value="Other">Other</option>
                </select>
              </div>
            </div>
            <div class="form-group">
              <label>Contact Phone</label>
              <input v-model="studentForm.contact_phone" type="tel" />
            </div>
            <div class="modal-actions">
              <button type="button" @click="closeStudentModal" class="btn-cancel">Cancel</button>
              <button type="submit" class="btn-save">Save Changes</button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Instructor Modal -->
    <div v-if="showInstructorModal" class="modal-overlay" @click="closeInstructorModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>Edit Instructor</h2>
          <button @click="closeInstructorModal" class="btn-close">×</button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="saveInstructor">
            <div class="form-group">
              <label>Instructor ID</label>
              <input v-model="instructorForm.instructor_id" disabled />
            </div>
            <div class="form-group">
              <label>Name *</label>
              <input v-model="instructorForm.name" required />
            </div>
            <div class="form-group">
              <label>Email *</label>
              <input v-model="instructorForm.email" type="email" required />
            </div>
            <div class="form-row">
              <div class="form-group">
                <label>Department *</label>
                <select v-model.number="instructorForm.dept_id" required>
                  <option value="">Select...</option>
                  <option v-for="dept in departments" :key="dept.id" :value="dept.id">
                    {{ dept.dept_name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <label>Title</label>
                <input v-model="instructorForm.title" />
              </div>
            </div>
            <div class="form-group">
              <label>Office Location</label>
              <input v-model="instructorForm.office_location" />
            </div>
            <div class="form-group">
              <label>Office Hours</label>
              <input v-model="instructorForm.office_hours" />
            </div>
            <div class="form-group">
              <label>Contact Phone</label>
              <input v-model="instructorForm.contact_phone" type="tel" />
            </div>
            <div class="modal-actions">
              <button type="button" @click="closeInstructorModal" class="btn-cancel">Cancel</button>
              <button type="submit" class="btn-save">Save Changes</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import { useAuthStore } from '@/components/stores/auth';
import { useRouter } from 'vue-router';

const authStore = useAuthStore();
const router = useRouter();

// Check if user is admin
if (!authStore.isAdmin) {
  router.push('/dashboard');
}

// State
const activeTab = ref('courses');
const loading = ref(false);
const error = ref(null);

// Data
const courses = ref([]);
const students = ref([]);
const instructors = ref([]);
const departments = ref([]);

// Modals
const showCourseModal = ref(false);
const showStudentModal = ref(false);
const showInstructorModal = ref(false);

// Forms
const editingCourse = ref(null);
const courseForm = ref({
  course_code: '',
  course_name: '',
  description: '',
  credits: 3,
  dept_id: '',
  course_type: ''
});

const studentForm = ref({});
const instructorForm = ref({});

// Methods
const loadData = async () => {
  loading.value = true;
  error.value = null;
  
  try {
    if (activeTab.value === 'courses') {
      const res = await fetch('http://localhost:3000/api/courses', {
        headers: { 'Authorization': `Bearer ${authStore.token}` }
      });
      courses.value = await res.json();
    } else if (activeTab.value === 'students') {
      const res = await fetch('http://localhost:3000/api/students', {
        headers: { 'Authorization': `Bearer ${authStore.token}` }
      });
      students.value = await res.json();
    } else if (activeTab.value === 'instructors') {
      const res = await fetch('http://localhost:3000/api/instructors', {
        headers: { 'Authorization': `Bearer ${authStore.token}` }
      });
      instructors.value = await res.json();
    }
  } catch (err) {
    error.value = 'Failed to load data';
  } finally {
    loading.value = false;
  }
};

const loadDepartments = async () => {
  try {
    const res = await fetch('http://localhost:3000/api/departments');
    departments.value = await res.json();
  } catch (err) {
    console.error('Failed to load departments:', err);
  }
};

const openCourseModal = (course = null) => {
  editingCourse.value = course;
  if (course) {
    courseForm.value = { ...course };
  } else {
    courseForm.value = {
      course_code: '',
      course_name: '',
      description: '',
      credits: 3,
      dept_id: '',
      course_type: ''
    };
  }
  showCourseModal.value = true;
};

const closeCourseModal = () => {
  showCourseModal.value = false;
  editingCourse.value = null;
};

const saveCourse = async () => {
  try {
    const url = editingCourse.value 
      ? `http://localhost:3000/api/courses/${editingCourse.value.id}`
      : 'http://localhost:3000/api/courses';
    
    const method = editingCourse.value ? 'PUT' : 'POST';
    
    const res = await fetch(url, {
      method,
      headers: {
        'Authorization': `Bearer ${authStore.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(courseForm.value)
    });
    
    if (res.ok) {
      alert(editingCourse.value ? 'Course updated!' : 'Course created!');
      closeCourseModal();
      loadData();
    } else {
      const data = await res.json();
      alert(data.message || 'Failed to save course');
    }
  } catch (err) {
    alert('Error saving course');
  }
};

const deleteCourse = async (course) => {
  if (!confirm(`Delete course ${course.course_code}?`)) return;
  
  try {
    const res = await fetch(`http://localhost:3000/api/courses/${course.id}`, {
      method: 'DELETE',
      headers: { 'Authorization': `Bearer ${authStore.token}` }
    });
    
    if (res.ok) {
      alert('Course deleted!');
      loadData();
    } else {
      alert('Failed to delete course');
    }
  } catch (err) {
    alert('Error deleting course');
  }
};

const openStudentModal = (student) => {
  studentForm.value = { ...student };
  showStudentModal.value = true;
};

const closeStudentModal = () => {
  showStudentModal.value = false;
};

const saveStudent = async () => {
  try {
    const res = await fetch(`http://localhost:3000/api/students/${studentForm.value.id}`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${authStore.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(studentForm.value)
    });
    
    if (res.ok) {
      alert('Student updated!');
      closeStudentModal();
      loadData();
    } else {
      const data = await res.json();
      alert(data.message || 'Failed to update student');
    }
  } catch (err) {
    alert('Error updating student');
  }
};

const openInstructorModal = (instructor) => {
  instructorForm.value = { ...instructor };
  showInstructorModal.value = true;
};

const closeInstructorModal = () => {
  showInstructorModal.value = false;
};

const saveInstructor = async () => {
  try {
    const res = await fetch(`http://localhost:3000/api/instructors/${instructorForm.value.id}`, {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${authStore.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(instructorForm.value)
    });
    
    if (res.ok) {
      alert('Instructor updated!');
      closeInstructorModal();
      loadData();
    } else {
      const data = await res.json();
      alert(data.message || 'Failed to update instructor');
    }
  } catch (err) {
    alert('Error updating instructor');
  }
};

const formatType = (type) => {
  const types = {
    'general_required': 'General Required',
    'major_required': 'Major Required',
    'major_elective': 'Major Elective',
    'university_elective': 'University Elective',
    'practical': 'Practical'
  };
  return types[type] || type;
};

// Watch tab changes
watch(activeTab, () => {
  loadData();
});

// Load initial data
onMounted(() => {
  loadDepartments();
  loadData();
});
</script>

<style scoped>
.admin-panel {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
}

.admin-header {
  text-align: center;
  margin-bottom: 2rem;
}

.admin-header h1 {
  font-size: 2.5rem;
  color: #1f2937;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #6b7280;
  font-size: 1.1rem;
}

.tab-nav {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 2rem;
  background: white;
  padding: 0.5rem;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.tab-btn {
  flex: 1;
  padding: 1rem;
  background: transparent;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  color: #6b7280;
}

.tab-btn:hover {
  background: #f3f4f6;
}

.tab-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.tab-content {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.content-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.content-header h2 {
  color: #1f2937;
  font-size: 1.8rem;
}

.btn-create {
  padding: 0.75rem 1.5rem;
  background: #10b981;
  color: white;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-create:hover {
  background: #059669;
  transform: translateY(-1px);
}

.data-table {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f9fafb;
}

th {
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  color: #374151;
  border-bottom: 2px solid #e5e7eb;
}

td {
  padding: 1rem;
  border-bottom: 1px solid #e5e7eb;
  color: #4b5563;
}

tbody tr:hover {
  background: #f9fafb;
}

.type-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
}

.type-badge.general_required { background: #dbeafe; color: #1e40af; }
.type-badge.major_required { background: #fee2e2; color: #991b1b; }
.type-badge.major_elective { background: #fef3c7; color: #92400e; }
.type-badge.university_elective { background: #d1fae5; color: #065f46; }
.type-badge.practical { background: #e9d5ff; color: #6b21a8; }

.actions {
  display: flex;
  gap: 0.5rem;
}

.btn-edit, .btn-delete {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-edit {
  background: #fef3c7;
  color: #92400e;
}

.btn-edit:hover {
  background: #fde68a;
}

.btn-delete {
  background: #fee2e2;
  color: #991b1b;
}

.btn-delete:hover {
  background: #fecaca;
}

.loading, .error {
  text-align: center;
  padding: 3rem;
  color: #6b7280;
}

.error {
  color: #ef4444;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal-content {
  background: white;
  border-radius: 12px;
  max-width: 600px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #e5e7eb;
}

.modal-header h2 {
  color: #1f2937;
  font-size: 1.5rem;
}

.btn-close {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: #6b7280;
  line-height: 1;
}

.modal-body {
  padding: 1.5rem;
}

.form-group {
  margin-bottom: 1.25rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #374151;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e5e7eb;
  border-radius: 6px;
  font-size: 1rem;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #667eea;
}

.form-group input:disabled {
  background: #f3f4f6;
  cursor: not-allowed;
}

.modal-actions {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
  padding-top: 1rem;
  border-top: 1px solid #e5e7eb;
}

.btn-cancel, .btn-save {
  flex: 1;
  padding: 0.75rem;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-cancel {
  background: #f3f4f6;
  color: #374151;
}

.btn-cancel:hover {
  background: #e5e7eb;
}

.btn-save {
  background: #667eea;
  color: white;
}

.btn-save:hover {
  background: #5568d3;
}

@media (max-width: 768px) {
  .admin-panel {
    padding: 1rem;
  }

  .form-row {
    grid-template-columns: 1fr;
  }

  .actions {
    flex-direction: column;
  }
}
</style>
