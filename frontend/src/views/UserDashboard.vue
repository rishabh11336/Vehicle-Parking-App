<template>
  <div>
    <Navbar />

    <div class="container my-5">
      <div v-if="isLoading" class="text-center py-5">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>
      <div v-else-if="errorMessage" class="alert alert-danger">{{ errorMessage }}</div>
      
      <div v-else class="row gy-4">
        <div class="col-lg-5 col-md-12">
          <div class="card p-4 h-100">
            <h4 class="fw-bold mb-4 text-center text-md-start">My Profile</h4>
            
            <div class="text-center mb-4">
              <img :src="editForm.image || user.image" alt="Profile Image" class="profile-image rounded-circle mb-2" />
              <div v-if="editing">
                <label for="imageUpload" class="btn btn-sm btn-outline-secondary">Change Photo</label>
                <input type="file" id="imageUpload" class="d-none" @change="handleImageUpload" />
              </div>
            </div>

            <div>
              <div v-if="editing" class="px-2">
                <div class="mb-3">
                  <label class="form-label small fw-bold">Full Name</label>
                  <input type="text" class="form-control" v-model="editForm.full_name" />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold">Email</label>
                  <input type="email" class="form-control" v-model="editForm.email" disabled />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold">Phone</label>
                  <input type="text" class="form-control" v-model="editForm.phone" />
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold">Address</label>
                  <textarea class="form-control" v-model="editForm.address" rows="2"></textarea>
                </div>
                <div class="mb-3">
                  <label class="form-label small fw-bold">Pin Code</label>
                  <input type="text" class="form-control" v-model="editForm.pin_code" />
                </div>
                <div class="d-grid gap-2 d-md-flex justify-content-md-end mt-4">
                    <button class="btn btn-secondary btn-sm" @click="toggleEdit">Cancel</button>
                    <button class="btn btn-primary btn-sm" @click="saveProfile">Save Changes</button>
                </div>
              </div>

              <div v-else>
                <ul class="list-group list-group-flush">
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap">
                    <strong class="text-muted">Name:</strong>
                    <span class="user-data-value">{{ user.full_name }}</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap">
                    <strong class="text-muted">Email:</strong>
                    <span class="user-data-value">{{ user.email }}</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap">
                    <strong class="text-muted">Phone:</strong>
                    <span class="user-data-value">{{ user.phone || 'Not provided' }}</span>
                  </li>
                   <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap">
                    <strong class="text-muted">Address:</strong>
                    <span class="text-end user-data-value">{{ user.address || 'Not provided' }}</span>
                  </li>
                   <li class="list-group-item d-flex justify-content-between align-items-center flex-wrap">
                    <strong class="text-muted">Pin Code:</strong>
                    <span class="user-data-value">{{ user.pin_code || 'Not provided' }}</span>
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
          <div class="card p-4 mb-4">
            <h5 class="fw-bold mb-3">Current Bookings</h5>
            <div v-if="currentBookings.length > 0">
              <ul class="list-group list-group-flush">
                <li v-for="booking in currentBookings" :key="booking.id" class="list-group-item px-0 py-3">
                  <div class="d-flex justify-content-between align-items-center">
                    <div>
                      <p class="mb-1"><strong>Location:</strong> {{ booking.lot_info ? booking.lot_info.lot_name : 'N/A' }}</p>
                      <small class="text-muted"><strong>On:</strong> {{ formatDate(booking.start_time) }}</small>
                    </div>
                    <div class="text-end">
                      <button class="btn btn-primary btn-sm" @click="viewBookingDetails(booking.id)">
                        View Details
                      </button>
                    </div>
                  </div>
                </li>
              </ul>
            </div>
            <div v-else class="text-center text-muted py-3">
                <p>You have no active bookings.</p>
            </div>
          </div>

          <div class="card p-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="fw-bold mb-0">Booking History</h5>
              <button class="btn btn-outline-secondary btn-sm">Download Full History</button>
            </div>
            <ul class="list-group list-group-flush">
              <li v-for="booking in bookingHistory" :key="booking.id"
                  class="list-group-item d-flex justify-content-between align-items-center px-0">
                <div>
                  <strong class="d-block">{{ booking.lot_info ? booking.lot_info.lot_name : 'N/A' }}</strong>
                  <small class="text-muted">{{ formatDate(booking.start_time) }}</small>
                </div>
                <span class="badge bg-primary rounded-pill price-badge">₹{{ booking.total_cost }}</span>
              </li>
            </ul>
             <div v-if="bookingHistory.length === 0" class="text-center text-muted py-3">
                <p>Your booking history is empty.</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <Footer />
  </div>
</template>

<script>
import Navbar from '../components/Navbar.vue'
import Footer from '../components/Footer.vue'
import axios from 'axios';

export default {
  name: 'UserProfile',
  components: {
    Navbar,
    Footer
  },
  data() {
    return {
      editing: false,
      isLoading: false,
      errorMessage: '',
      // User object will be filled by the API call
      user: {},
      // This form object will be used for editing
      editForm: {},
      // These arrays will be filled by the API call
      currentBookings: [],
      bookingHistory: []
    };
  },
  methods: {
    async fetchDashboardData() {
        this.isLoading = true;
        this.errorMessage = '';
        try {
            const token = localStorage.getItem('authToken');
            if (!token) {
                this.$router.push('/login');
                return;
            }

            const profilePromise = axios.get('http://127.0.0.1:5000/user_profile', {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            const bookingsPromise = axios.get('http://127.0.0.1:5000/user_bookings', {
                headers: { 'Authorization': `Bearer ${token}` }
            });

            const [profileResponse, bookingsResponse] = await Promise.all([profilePromise, bookingsPromise]);
            
            this.user = profileResponse.data;
            // Generate a placeholder image if no actual image URL is available
            this.user.image = `https://placehold.co/150x150/5E8B7E/FFFFFF?text=${profileResponse.data.full_name.charAt(0).toUpperCase()}`;

            const allBookings = bookingsResponse.data;
            // Filter current bookings based on 'Active' or 'Parked' status
            this.currentBookings = allBookings.filter(b => b.status === 'Active' || b.status === 'Parked');
            // Filter booking history based on 'Completed' or 'canceled' status
            this.bookingHistory = allBookings.filter(b => b.status === 'Completed' || b.status === 'canceled');

        } catch (error) {
            this.errorMessage = "Failed to load dashboard data. Please try again.";
            console.error("Error fetching dashboard data:", error);
        } finally {
            this.isLoading = false;
        }
    },
    toggleEdit() {
      this.editing = !this.editing;
      if (this.editing) {
          // Create a shallow copy of user data for editing
          this.editForm = { ...this.user };
          // Ensure image is also copied for editing, or handled separately if it's a file input
          if (!this.editForm.image) {
            this.editForm.image = `https://placehold.co/150x150/5E8B7E/FFFFFF?text=${this.user.full_name.charAt(0).toUpperCase()}`;
          }
      }
    },
    async saveProfile() {
        this.isLoading = true;
        this.errorMessage = '';
        try {
            const token = localStorage.getItem('authToken');
            // The /api/profile endpoint is typically for updating the current user's profile
            await axios.patch('http://127.0.0.1:5000/edit_user_profile', this.editForm, {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            // Update the main user object after successful save
            this.user = { ...this.editForm };
            this.editing = false;
        } catch (error) {
            this.errorMessage = "Failed to save profile changes. Please check your input.";
            console.error("Error saving profile:", error);
        } finally {
            this.isLoading = false;
        }
    },
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => { this.editForm.image = e.target.result; };
        reader.readAsDataURL(file);
        // You might want to upload this file to a server here instead of just displaying it
        // For now, this just updates the local preview.
      }
    },
    viewBookingDetails(bookingId) {
      // Navigate to a specific booking details page
      this.$router.push(`/user/booking-details/${bookingId}`);
    },
    formatDate(isoString) {
        if (!isoString) return 'N/A';
        // Parse the ISO string to a Date object
        const date = new Date(isoString);
        // Define options for formatting
        const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit', hour12: true };
        // Format the date to a readable string
        return date.toLocaleDateString(undefined, options);
    }
  },
  // The created() hook is the perfect place to fetch initial data
  created() {
      this.fetchDashboardData();
  }
};
</script>