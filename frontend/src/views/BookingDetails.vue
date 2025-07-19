<template>
  <div>
    <Navbar />

    <div class="container my-5">
      <!-- BACK TO PROFILE BUTTON -->
      <div class="mb-4">
        <router-link to="/user/dashboard" class="text-decoration-none text-primary fw-semibold">
          <i class="bi bi-arrow-left-circle me-2"></i>Back to My Dashboard
        </router-link>
      </div>
      
      <div v-if="booking">
        <div class="row g-4">
          <!-- BOOKING INFO & STATUS        -->
          <div class="col-lg-8">
            <div class="card p-4 h-100">
              <div class="d-flex justify-content-between align-items-start mb-3">
                <div>
                  <h3 class="fw-bold mb-1">{{ booking.location }}</h3>
                  <p class="text-muted mb-0"><i class="bi bi-geo-alt-fill me-1"></i>{{ booking.address }}</p>
                </div>
                <!-- BADGE COLOR CHANGES BASED ON STATUS -->
                <span :class="`badge fs-6 ${getStatusClass(booking.status)}`">{{ booking.status }}</span>
              </div>
              <hr>
              
              <!-- BOOKING DETAILS -->
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

              <!-- COST SUMMARY -->
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
                    <span>₹{{ booking.totalCost.toFixed(2) }}</span>
                  </li>
                </ul>
              </div>
            </div>
          </div>
          <!--  ACTIONS  COLUMN  -->
          <div class="col-lg-4">
            <div class="card p-4">
              <h5 class="fw-semibold mb-3">Manage Booking</h5>
              <div class="d-grid gap-2">
                <!-- CHANGE BOOKING BUTTON -->
                <button class="btn btn-outline-primary" @click="handleAction('change')" :disabled="booking.status !== 'Active'">
                  <i class="bi bi-pencil-square me-2"></i>Change Booking
                </button>
                <!-- CANCEL BOOKING BUTTON -->
                <button class="btn btn-outline-danger" @click="handleAction('cancel')" :disabled="booking.status !== 'Active'">
                  <i class="bi bi-x-circle me-2"></i>Cancel Booking
                </button>
                <!-- VACATE BOOKING BUTTON -->
                <button class="btn btn-outline-warning text-dark" @click="handleAction('vacate')" :disabled="booking.status !== 'Parked'">
                  <i class="bi bi-car-front-fill me-2"></i>Vacate Parking
                </button>
                <!-- RELEASE BOOKING BUTTON -->
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

export default {
  name: 'BookingDetailsPage',
  components: {
    Navbar,
    Footer
  },
  data() {
    return {
      booking: null,
      // DUMMY DATA
      allBookings: [
        { 
          id: 101, 
          location: 'Phoenix Mall Parking', 
          address: 'Viman Nagar, Pune',
          vehicleNumber: 'MH 12 AB 1234',
          startDate: '2025-06-18', startTime: '13:00',
          endDate: '2025-06-18', endTime: '16:00',
          rateApplied: 80,
          duration: '3.0 Hours',
          totalCost: 240,
          status: 'Active', 
        },
        { 
          id: 102, 
          location: 'Airport T2 Parking', 
          address: 'Lohegaon Airport, Pune',
          vehicleNumber: 'MH 14 CD 5678',
          startDate: '2025-06-20', startTime: '10:30',
          endDate: '2025-06-22', endTime: '18:30',
          rateApplied: 65,
          duration: '3 Days (8 hrs/day)',
          totalCost: 1560,
          status: 'Parked', 
        },
        {
          id: 1, 
          location: 'Pune Station Lot',
          address: 'Near Main Entrance, Pune Station',
          vehicleNumber: 'MH 12 AB 1234',
          startDate: '2025-05-25', startTime: '10:30',
          endDate: '2025-05-25', endTime: '12:30',
          rateApplied: 50,
          duration: '2.0 Hours',
          totalCost: 100,
          status: 'Completed', 
        }
      ]
    };
  },
  created() {
    const bookingId = parseInt(this.$route.params.id);
    
    if (bookingId) {
      this.booking = this.allBookings.find(b => b.id === bookingId) || null;
    } else {
      this.booking = this.allBookings[1];
    }
  },
  methods: {
      handleAction(actionType) {
        // THIS FUNCTION CHECKS THE BOOKING STATUS BEFORE PERFORMING ANY ACTION.
        
        switch(actionType) {
            case 'change':
            case 'cancel':
                // CHANGE/CANCEL IS ONLY ALLOWED IF THE BOOKING IS ACTIVE.
                if(this.booking.status !== 'Active') {
                    alert("Action not allowed. Vehicle is already parked or booking is completed.");
                } else {
                    alert(`'${actionType}' action initiated.`);
                }
                break;
            case 'vacate':
                // VACATE IS ONLY ALLOWED IF THE VEHICLE IS PARKED.
                if(this.booking.status === 'Parked') {
                    alert("Vehicle has been marked as vacated. You can now release the booking.");
                    this.booking.status = 'Vacated'; 
                }
                break;
            case 'release':
                // RELEASE IS ONLY ALLOWED AFTER THE VEHICLE HAS BEEN 'VACATED'.
                if(this.booking.status !== 'Vacated') {
                    alert("Action not allowed. Please vacate the parking spot first.");
                } else {
                    alert("Parking released and payment processed successfully!");
                    this.booking.status = 'Completed';
                }
                break;
        }
      },
      // CHANGE COLOR BASED ON PARKING STATUS
      getStatusClass(status) {
          if (status === 'Active') return 'bg-info text-dark';
          if (status === 'Parked') return 'bg-success';
          if (status === 'Vacated') return 'bg-warning text-dark'; 
          if (status === 'Completed') return 'bg-secondary';
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
