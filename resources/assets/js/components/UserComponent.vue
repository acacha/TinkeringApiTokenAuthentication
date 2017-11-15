<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <div class="panel panel-default">
                    <div class="panel-heading">User</div>

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
          user : {}
        }
      },
      mounted() {
        console.log('Component mounted.')
        axios.get('/api/user').then( response => {
          this.user = response.data
          console.log(response.data)
        }).catch( error => {
          console.log(error)
        })
      }
    }
</script>
