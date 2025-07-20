<template>
  <div>
    <Navbar />

    <div class="container my-5">
      <div class="mb-4">
        <router-link to="/user/dashboard" class="text-decoration-none text-primary fw-semibold">
          <i class="bi bi-arrow-left-circle me-2"></i>Back to My Dashboard
        </router-link>
      </div>
      
      <div v-if="booking">
        <div class="row g-4">
          <div class="col-lg-8">
            <div class="card p-4 h-100">
              <div class="d-flex justify-content-between align-items-start mb-3">
                <div>
                  <h3 class="fw-bold mb-1">{{ booking.location }}</h3>
                  <p class="text-muted mb-0"><i class="bi bi-geo-alt-fill me-1"></i>{{ booking.address }}</p>
                </div>
                <span :class="`badge fs-6 ${getStatusClass(booking.status)}`">{{ booking.status }}</span>
              </div>
              <hr>
              
              <div class="row">
                <div class="col-sm-6 mb-3">
                  <strong class="text-muted d-block">Vehicle Number</strong>
                  <span>{{ booking.vehicleNumber }}</span>
                </div>
                <div class="col-sm-6 mb-3">
                  <strong class="text-muted d-block">Booking ID</strong>
                  <span>#{{ booking.id }}</span>
                </div>
                <div class="col-sm-6 mb-3">
                  <strong class="text-muted d-block">Start Date & Time</strong>
                  <span>{{ booking.startDate }} at {{ booking.startTime }}</span>
                </div>
                <div class="col-sm-6 mb-3">
                  <strong class="text-muted d-block">End Date & Time</strong>
                  <span>{{ booking.endDate }} at {{ booking.endTime }}</span>
                </div>
              </div>

              <div class="mt-3">
                <h5 class="fw-semibold">Cost Summary</h5>
                <ul class="list-group list-group-flush">
                   <li class="list-group-item d-flex justify-content-between px-0">
                    <span>Rate</span>
                    <span>₹{{ booking.rateApplied.toFixed(2) }}/hr</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between px-0">
                    <span>Duration</span>
                    <span>{{ booking.duration }}</span>
                  </li>
                  <li class="list-group-item d-flex justify-content-between px-0 fw-bold">
                    <span>Total Cost</span>
                    <span v-if="booking.status === 'canceled'">Cancelled</span>
                    <span v-else>₹{{ booking.totalCost.toFixed(2) }}</span>
                  </li>
                </ul>
              </div>
            </div>
          </div>
          <div class="col-lg-4">
            <div class="card p-4">
              <h5 class="fw-semibold mb-3">Manage Booking</h5>
              <div class="d-grid gap-2">
                <button class="btn btn-outline-primary" @click="handleAction('change')" :disabled="booking.status !== 'Active'">
                  <i class="bi bi-pencil-square me-2"></i>Change Booking
                </button>
                <button class="btn btn-outline-danger" @click="handleAction('cancel')" :disabled="booking.status !== 'Active'">
                  <i class="bi bi-x-circle me-2"></i>Cancel Booking
                </button>
                <button class="btn btn-outline-warning text-dark" @click="handleAction('vacate')" :disabled="booking.status !== 'Parked'">
                  <i class="bi bi-car-front-fill me-2"></i>Vacate Parking
                </button>
                <button class="btn btn-success" @click="handleAction('release')" :disabled="booking.status !== 'Vacated'">
                  <i class="bi bi-box-arrow-right me-2"></i>Release & Pay
                </button>
              </div>
              <p v-if="booking.status === 'Completed'" class="text-muted small mt-3 text-center">This booking is completed.</p>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="text-center py-5">
        <p class="text-muted">Booking not found or an error occurred.</p>
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
  name: 'BookingDetailsPage',
  components: {
    Navbar,
    Footer
  },
  data() {
    return {
      booking: null,
    };
  },
  async created() {
    const bookingId = parseInt(this.$route.params.id);
    if (bookingId) {
      await this.fetchBookingDetails(bookingId);
    }
  },
  methods: {
    async fetchBookingDetails(id) {
      try {
        const token = localStorage.getItem('authToken'); 
        if (!token) {
          console.error('Bearer token not found in local storage.');
          // Redirect to login or handle authentication error
          return;
        }

        const response = await axios.get(`http://127.0.0.1:5000/user/booking-details/${id}`, {
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });
        const apiBooking = response.data[0]; 

        
        this.booking = {
          id: apiBooking.id,
          location: apiBooking.lot_info.lot_name,
          address: apiBooking.lot_info.address,
          vehicleNumber: apiBooking.vehicle_number,
          startDate: this.formatDate(apiBooking.start_time),
          startTime: this.formatTime(apiBooking.start_time),
          endDate: this.formatDate(apiBooking.end_time),
          endTime: this.formatTime(apiBooking.end_time),
          rateApplied: apiBooking.lot_info.price_per_hour,
          duration: this.calculateDuration(apiBooking.start_time, apiBooking.end_time, apiBooking.booking_type),
          totalCost: apiBooking.total_cost,
          status: apiBooking.status,
        };
      } catch (error) {
        console.error('Error fetching booking details:', error);
        this.booking = null; 
      }
    },
    formatDate(dateTimeString) {
      if (!dateTimeString) return 'N/A';
      const date = new Date(dateTimeString);
      return date.toLocaleDateString('en-CA'); 
    },
    formatTime(dateTimeString) {
      if (!dateTimeString) return 'N/A';
      const date = new Date(dateTimeString);
      return date.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: false });
    },
    calculateDuration(startTime, endTime, bookingType) {
      if (!startTime || !endTime) return 'N/A';

      const start = new Date(startTime);
      const end = new Date(endTime);
      const diffMs = end - start; 

      if (bookingType === 'hourly') {
        const diffHours = diffMs / (1000 * 60 * 60);
        return `${diffHours.toFixed(1)} Hours`;
      } else if (bookingType === 'daily') {
        const diffDays = diffMs / (1000 * 60 * 60 * 24);
        return `${Math.ceil(diffDays)} Days`; 
      }
      return 'N/A';
    },
    async handleAction(actionType) { // Made async to await API calls
      switch(actionType) {
          case 'change':
              if(this.booking.status !== 'Active') {
                  alert("Action not allowed. Vehicle is already parked or booking is completed.");
              } else {
                  alert(`'${actionType}' action initiated.`);
                  // Here you would typically navigate to a change booking page
              }
              break;
          case 'cancel':
              if(this.booking.status !== 'Active') {
                  alert("Action not allowed. Booking is not active.");
              } else {
                  if (confirm("Are you sure you want to cancel this booking?")) {
                      try {
                          const token = localStorage.getItem('authToken');
                          if (!token) {
                              console.error('Bearer token not found in local storage.');
                              alert('Authentication error. Please log in again.');
                              return;
                          }
                          const bookingIdToCancel = this.booking.id;
                          const response = await axios.patch(`http://127.0.0.1:5000/cancel_booking/${bookingIdToCancel}`, {}, {
                              headers: {
                                  'Authorization': `Bearer ${token}`
                              }
                          });
                          if (response.status === 200) {
                              alert("Booking cancelled successfully!");
                              // Update the status locally or refetch booking details
                              this.booking.status = 'canceled'; // Set to 'canceled' (lowercase)
                              // Optionally, re-fetch booking details to ensure state is synchronized with backend
                              // await this.fetchBookingDetails(bookingIdToCancel); 
                          } else {
                              alert("Failed to cancel booking. Please try again.");
                          }
                      } catch (error) {
                          console.error('Error cancelling booking:', error);
                          alert('An error occurred while trying to cancel the booking.');
                      }
                  }
              }
              break;
          case 'vacate':
              if(this.booking.status === 'Parked') {
                  alert("Vehicle has been marked as vacated. You can now release the booking.");
                  this.booking.status = 'Vacated'; 
                  // Here you would typically make an API call to update the booking status to 'Vacated'
              } else {
                  alert("Action not allowed. Vehicle is not marked as parked.");
              }
              break;
          case 'release':
              if(this.booking.status !== 'Vacated') {
                  alert("Action not allowed. Please vacate the parking spot first.");
              } else {
                  alert("Parking released and payment processed successfully!");
                  this.booking.status = 'Completed';
                  // Here you would typically make an API call to finalize the booking and process payment
              }
              break;
      }
    },
    getStatusClass(status) {
        if (status === 'Active') return 'bg-info text-dark';
        if (status === 'Parked') return 'bg-success';
        if (status === 'Vacated') return 'bg-warning text-dark'; 
        if (status === 'Completed') return 'bg-secondary';
        if (status === 'canceled') return 'bg-danger'; // Changed to 'canceled' (lowercase)
        return 'bg-dark';
    }
  }
};
</script>

<style scoped>
.card {
  border: 1px solid #e9ecef;
  border-radius: 0.75rem;
}
.list-group-item {
    background-color: transparent;
    padding-left: 0;
    padding-right: 0;
}
</style>