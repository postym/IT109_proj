<!-- <template>
  <v-container fluid id="cont"class="fill-height">
    <v-row justify="center" align="center">
      <v-col cols="12" sm="6" md="4">
        <v-card class="pa-6" elevation="10">
          
          <v-card-title class="text-h5 text-center">
            LogIn As:
          </v-card-title>


        
          <v-card-actions class="justify-center">
            <v-btn
              color="primary"
              class="ma-2"
              @click="loginAsCustomer"
            >
              Customer
            </v-btn>

            <v-btn
              color="secondary"
              class="ma-2"
              @click="loginAsAdmin"
            >
              Admin
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template> -->

<template>
  <v-container fluid id="cont" class="fill-height">
    <v-row justify="center" align="center">
      <v-col cols="12" sm="10" md="8">
        <v-card class="pa-6 user-card" elevation="10">
          <v-row>
            <!-- Left Column: Welcome Address with Photo -->
            <v-col cols="12" md="6" class="welcome-column text-center">
              <v-img
                src="@/assets/welcome.jfif"
                alt="Welcome Image"
                contain
                class="mb-4"
                height="200"
/>
              <v-card-title class="text-h5 white--text">
                안녕하세요!
              </v-card-title>
              <v-card-title class="text-h5 white--text">
                Welcome to K-Bite Lounge!
              </v-card-title>
              <v-card-text class="white--text">
                Experience the authentic flavors of Korea! Let the vibrant world of Korean cuisine delight your senses.
              </v-card-text>
            </v-col>

            <!-- Right Column: User Login -->
            <br><br><br>
            <v-col cols="12" md="6" class="login-column">
              <v-card-title class="text-h5 text-center white--text">
               Login as: 
              </v-card-title>
              <br><br><br>
              <v-card-actions class="justify-center">
                <v-btn
                  class="ma-2 gradient-button"
                  elevation="5"
                  @click="loginAsCustomer"
                >
                  Customer
                </v-btn>

                <v-btn
                  class="ma-2 gradient-button"
                  elevation="5"
                  @click="loginAsAdmin"
                >
                  Admin
                </v-btn>
              </v-card-actions>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { supabase } from '@/lib/supabase';
import { useRouter } from 'vue-router';

export default {
  name: 'UserLogin',
  setup() {
    const router = useRouter();

    const loginAsCustomer = async () => {
      const { data, error } = await supabase
        .from('users')
        .select('*')
        .eq('id', 1);
      if (error) {
        console.error('Error fetching user:', error);
      } else {
        console.log(data[0].id);
        localStorage.setItem('userId', data[0].id);
        router.push('/shop'); // Navigate to shop route
      }
    };

    const loginAsAdmin = async () => {
      const { data, error } = await supabase
        .from('users')
        .select('*')
        .eq('id', 2);
      if (error) {
        console.error('Error fetching user:', error);
      } else {
        console.log(data[0].id);
        localStorage.setItem('userId', data[0].id);
        router.push('/dashboard'); // Navigate to dashboard route
      }
    };

    return {
      loginAsCustomer,
      loginAsAdmin,
    };
  },
};
</script>

<style>

 /* From Uiverse.io by kandalgaonkarshubham */
#cont {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #3854b8, #ca9af2);
  display: flex;
  align-items: center;
  justify-content: center;
  /* --s: 37px; 

  --c: #0000, #282828 0.5deg 119.5deg, #0000 120deg;
  --g1: conic-gradient(from 60deg at 56.25% calc(425% / 6), var(--c));
  --g2: conic-gradient(from 180deg at 43.75% calc(425% / 6), var(--c));
  --g3: conic-gradient(from -60deg at 50% calc(175% / 12), var(--c));
  background: var(--g1), var(--g1) var(--s) calc(1.73 * var(--s)), var(--g2),
    var(--g2) var(--s) calc(1.73 * var(--s)), var(--g3) var(--s) 0,
    var(--g3) 0 calc(1.73 * var(--s)) #1e1e1e;
  background-size: calc(2 * var(--s)) calc(3.46 * var(--s)); */
}

/* Card Styling */
.user-card {
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2);
}

/* Welcome Column Styling */
.welcome-column {
  border-right: 1px solid rgba(255, 255, 255, 0.2);
  padding-right: 20px;
}

/* Text Styling */
.v-card-title,
.v-card-text {
  font-weight: 600;
}

/* Button Styling */
.gradient-button {
  color: white;
  background: linear-gradient(135deg, #3854b8, #ca9af2);
  border: none;
  font-weight: bold;
  text-transform: uppercase;
  border-radius: 8px;
  padding: 10px 20px;
  transition: transform 0.2s, box-shadow 0.2s;
}

.gradient-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.2);
}


.v-card {
  border-radius: 12px;
}
</style>
