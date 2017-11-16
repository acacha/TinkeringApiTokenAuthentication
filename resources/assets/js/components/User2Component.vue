<template>
    <div class="container">
        <div class="row">
            <div class="alert alert-danger" v-if="error">
                <strong>Error!</strong> {{ error_message }}
            </div>
        </div>
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <div class="panel panel-default">
                    <div class="panel-heading">User 2. Authorized via Authorization Header. Axios request not authorized (API GUARD | Driver: token)</div>

                    <div class="panel-body">
                        {{ user }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    import axios from 'axios';

    export default {
      data() {
        return {
          user : {},
          error: false,
          error_message : ''
        }
      },
      mounted() {
        console.log('Component mounted.')
        axios.get('/api/user',{
          headers: {'Authorization': 'Bearer uxMz6qL36PVFdKcmNR1jGd3bAe5PEHmEFA9056yYxSmvpcsu8IqGRRCUHJRH'},
        }).then( response => {
          this.user = response.data
          console.log(response.data)
        }).catch( error => {
          this.error = true
          this.error_message = error.message
          console.log(error)
        })
      }
    }
</script>
