<template>
  <div>
    <Navbar />

    <div class="container my-5">
      <div v-if="isLoading" class="text-center py-5">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
        <p class="text-muted mt-3">Fetching your dashboard data...</p>
      </div>
      
      <div v-else-if="errorMessage" class="alert alert-danger text-center">{{ errorMessage }}</div>
      
      <div v-else class="row gy-4">
        <div class="col-lg-5 col-md-12">
          <div class="card p-4 h-100 shadow-sm">
            <h4 class="fw-bold mb-4 text-center text-md-start">My Profile</h4>
            
            <div class="text-center mb-4">
              <img v-if="user.image" :src="user.image" alt="Profile Image" class="profile-image rounded-circle mb-2 border border-3 border-light" />
              <div v-else class="profile-avatar-placeholder rounded-circle mb-2 mx-auto d-flex align-items-center justify-content-center border border-3 border-light">
                <span class="avatar-text">{{ user.full_name ? user.full_name.charAt(0).toUpperCase() : 'U' }}</span>
              </div>
              
              <div v-if="editing" class="mt-3">
                <label for="imageUpload" class="btn btn-sm btn-outline-primary">
                  <i class="bi bi-camera me-1"></i>Change Photo
                </label>
                <input type="file" id="imageUpload" class="d-none" @change="handleImageUpload" accept="image/*" />
              </div>
            </div>

            <div>
              <div v-if="editing" class="px-2">
                <div class="mb-3">
                  <label class="form-label small fw-bold text-muted">Full Name</label>
                  <input type="text" class="form-control" v-model="editForm.full_name" placeholder="Enter full name" />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold text-muted">Email</label>
                  <input type="email" class="form-control" v-model="editForm.email" disabled />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold text-muted">Phone</label>
                  <input type="tel" class="form-control" v-model="editForm.phone" placeholder="e.g., +91 1234567890" />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold text-muted">Address</label>
                  <textarea class="form-control" v-model="editForm.address" rows="2" placeholder="Enter your address"></textarea>
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold text-muted">Pin Code</label>
                  <input type="text" class="form-control" v-model="editForm.pincode" placeholder="Enter pincode" />
                </div>
                <div class="d-grid gap-2 d-md-flex justify-content-md-end mt-4">
                    <button class="btn btn-secondary btn-sm" @click="toggleEdit">Cancel</button>
                    <button class="btn btn-primary btn-sm" @click="saveProfile">Save Changes</button>
                </div>
              </div>

              <div v-else>
                <ul class="list-group list-group-flush border-top border-bottom">
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap py-2">
                    <strong class="text-muted">Name:</strong>
                    <span class="user-data-value text-end">{{ user.full_name || 'N/A' }}</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap py-2">
                    <strong class="text-muted">Email:</strong>
                    <span class="user-data-value text-end">{{ user.email || 'N/A' }}</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap py-2">
                    <strong class="text-muted">Phone:</strong>
                    <span class="user-data-value text-end">{{ user.phone || 'Not provided' }}</span>
                  </li>
                   <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap py-2">
                    <strong class="text-muted">Address:</strong>
                    <span class="text-end user-data-value flex-grow-1 ms-3">{{ user.address || 'Not provided' }}</span>
                  </li>
                   <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap py-2">
                    <strong class="text-muted">Pin Code:</strong>
                    <span class="user-data-value text-end">{{ user.pincode || 'Not provided' }}</span>
                  </li>
                </ul>
                <div class="text-center mt-4">
                  <button class="btn btn-primary w-100" @click="toggleEdit">Edit Profile</button>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="col-lg-7 col-md-12">
          <div class="card p-4 mb-4 shadow-sm">
            <h5 class="fw-bold mb-3">Current Bookings</h5>
            <div v-if="currentBookings.length > 0">
              <ul class="list-group list-group-flush border-top border-bottom">
                <li v-for="booking in currentBookings" :key="booking.id" class="list-group-item px-0 py-3">
                  <div class="d-flex justify-content-between align-items-center">
                    <div>
                      <p class="mb-1 fw-semibold">
                        <i class="bi bi-pin-map-fill me-1 text-primary"></i>
                        {{ booking.lot_info ? booking.lot_info.lot_name : 'N/A' }}
                      </p>
                      <small class="text-muted">
                        <i class="bi bi-calendar me-1"></i>
                        {{ formatBookingDateTime(booking.start_time) }}
                      </small>
                    </div>
                    <div class="text-end">
                      <router-link :to="`/user/booking-details/${booking.id}`" class="btn btn-primary btn-sm">
                        View Details
                      </router-link>
                    </div>
                  </div>
                </li>
              </ul>
            </div>
            <div v-else class="text-center text-muted py-3">
                <p><i class="bi bi-info-circle me-2"></i>You have no active bookings.</p>
                <router-link to="/user/parking" class="btn btn-outline-primary btn-sm">Find Parking</router-link>
            </div>
          </div>

          <div class="card p-4 shadow-sm">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="fw-bold mb-0">Booking History</h5>
              <button class="btn btn-outline-secondary btn-sm" :disabled="bookingHistory.length === 0">
                <i class="bi bi-download me-1"></i>Download Full History
              </button>
            </div>
            <ul class="list-group list-group-flush border-top border-bottom">
              <li v-for="booking in bookingHistory" :key="booking.id"
                  class="list-group-item d-flex justify-content-between align-items-center px-0 py-3">
                <div>
                  <strong class="d-block">{{ booking.lot_info ? booking.lot_info.lot_name : 'N/A' }}</strong>
                  <small class="text-muted">
                    <i class="bi bi-calendar me-1"></i>
                    {{ formatBookingDateTime(booking.start_time) }}
                  </small>
                </div>
                <div>
                  <span v-if="booking.status === 'canceled'" class="badge bg-danger rounded-pill px-3 py-2 fs-7">Cancelled</span>
                  <span v-else class="badge bg-success rounded-pill px-3 py-2 fs-7">₹{{ booking.total_cost ? booking.total_cost.toFixed(2) : '0.00' }}</span>
                </div>
              </li>
            </ul>
             <div v-if="bookingHistory.length === 0" class="text-center text-muted py-3">
                <p><i class="bi bi-info-circle me-2"></i>Your booking history is empty.</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <Footer />
  </div>
</template>

<script>
import Navbar from '../components/Navbar.vue';
import Footer from '../components/Footer.vue';
import axios from 'axios';

export default {
  name: 'UserDashboard',
  components: {
    Navbar,
    Footer
  },
  data() {
    return {
      editing: false,
      isLoading: false,
      errorMessage: '',
      user: {
        id: null,
        full_name: '',
        email: '',
        phone: '',
        address: '',
        pincode: '', // Changed from 'pin_code' to 'pincode' for consistency with API response structure
        image: '' // Placeholder for profile image URL
      },
      editForm: { // Used for temporary edits, copied from 'user'
        full_name: '',
        email: '',
        phone: '',
        address: '',
        pincode: '', // Changed to 'pincode'
        image: ''
      },
      currentBookings: [],
      bookingHistory: []
    };
  },
  async created() {
      // Fetch all data when the component is created
      await this.fetchDashboardData();
  },
  methods: {
    async fetchDashboardData() {
        this.isLoading = true;
        this.errorMessage = '';
        try {
            const token = localStorage.getItem('authToken');
            if (!token) {
                this.$router.push('/login'); // Redirect to login if not authenticated
                return;
            }

            // Fetch user profile
            const profileResponse = await axios.get('http://127.0.0.1:5000/user_profile', {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            this.user = { ...profileResponse.data }; // Copy data to user object
            // Ensure pincode is read correctly from API (if API uses 'pincode' instead of 'pin_code')
            if (this.user.pin_code && !this.user.pincode) {
                this.user.pincode = this.user.pin_code;
                delete this.user.pin_code; // Clean up if API has both
            }
            // Generate a placeholder avatar if no image URL is provided by the API
            if (!this.user.image) {
                // This would be replaced by actual image handling if your backend serves profile pictures
                // For now, it's just for display based on first letter of name
                this.user.image = ''; // Set to empty to use placeholder div
            }

            // Fetch all user bookings
            const bookingsResponse = await axios.get('http://127.0.0.1:5000/user_bookings', {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            
            const allBookings = bookingsResponse.data;

            // Filter bookings based on status
            this.currentBookings = allBookings.filter(b => b.status === 'Active' || b.status === 'Parked');
            this.bookingHistory = allBookings.filter(b => b.status === 'Completed' || b.status === 'canceled'); // 'canceled' (lowercase)

        } catch (error) {
            console.error("Error fetching dashboard data:", error);
            this.errorMessage = "Failed to load dashboard data. Please check your internet connection or try again later.";
            // Specific error handling for 401 Unauthorized
            if (error.response && error.response.status === 401) {
                this.$router.push('/login');
            }
        } finally {
            this.isLoading = false;
        }
    },
    toggleEdit() {
      this.editing = !this.editing;
      if (this.editing) {
          // Deep copy user data to editForm to avoid direct mutation
          this.editForm = JSON.parse(JSON.stringify(this.user));
      }
    },
    async saveProfile() {
        this.isLoading = true;
        this.errorMessage = '';
        try {
            const token = localStorage.getItem('authToken');
            
            // Only send fields that can be updated. Email is disabled.
            const dataToUpdate = {
                full_name: this.editForm.full_name,
                phone: this.editForm.phone,
                address: this.editForm.address,
                pincode: this.editForm.pincode // Changed to pincode
                // If you handle image uploads, you'd add image logic here (e.g., formData)
            };

            await axios.patch('http://127.0.0.1:5000/edit_user_profile', dataToUpdate, {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            
            // Re-fetch profile data to ensure local state is synchronized with backend
            // This is safer than manually updating `this.user = { ...this.editForm };`
            await this.fetchDashboardData(); 
            this.editing = false;
            alert("Profile updated successfully!"); // User feedback
        } catch (error) {
            console.error("Error saving profile:", error.response ? error.response.data : error.message);
            this.errorMessage = error.response && error.response.data && error.response.data.message 
                                ? error.response.data.message 
                                : "Failed to save profile changes. Please try again.";
        } finally {
            this.isLoading = false;
        }
    },
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
            this.editForm.image = e.target.result; // For immediate preview
            // In a real application, you'd upload this 'file' to your server
            // and then update `this.user.image` with the URL returned by the server.
            // Example: uploadImageToServer(file).then(url => this.user.image = url);
        };
        reader.readAsDataURL(file);
      }
    },
    viewBookingDetails(bookingId) {
      this.$router.push(`/user/booking-details/${bookingId}`);
    },
    formatBookingDateTime(isoString) {
        if (!isoString) return 'N/A';
        const date = new Date(isoString);
        // Format example: July 19, 2025 at 10:00 AM
        return date.toLocaleDateString(undefined, { year: 'numeric', month: 'long', day: 'numeric' }) + 
               ' at ' + 
               date.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit', hour12: true });
    }
  }
};
</script>

<style scoped>
.card {
  border: none; /* Removed default border */
  border-radius: 0.75rem;
  box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075); /* Subtle shadow for depth */
}

/* Profile Image / Avatar Styling */
.profile-image {
  width: 120px;
  height: 120px;
  object-fit: cover;
  border: 4px solid var(--bs-primary); /* Border using Bootstrap primary color */
  box-shadow: 0 0 0 3px rgba(var(--bs-primary-rgb), .25); /* Inner glow effect */
}

.profile-avatar-placeholder {
  width: 120px;
  height: 120px;
  background-color: var(--bs-primary); /* Bootstrap primary color */
  color: white;
  font-size: 4rem;
  font-weight: bold;
  display: flex; /* For centering text */
  align-items: center; /* For centering text */
  justify-content: center; /* For centering text */
  border: 4px solid var(--bs-primary);
  box-shadow: 0 0 0 3px rgba(var(--bs-primary-rgb), .25);
}

.list-group-item {
  background-color: transparent;
  padding-left: 0;
  padding-right: 0;
  border-color: #eee; /* Lighter border for list items */
}

.user-data-value {
    font-weight: 500;
    color: #343a40; /* Darker text for values */
}

.price-badge {
    min-width: 80px; /* Give some consistent width for badges */
}

/* Responsive adjustments */
@media (min-width: 768px) {
  .text-md-start {
    text-align: left !important;
  }
}
</style>