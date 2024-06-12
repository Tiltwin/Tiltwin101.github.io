// (function () {

$.ajaxSetup({
    type: "POST",
    data: {},
    dataType: 'json',
    xhrFields: {
       withCredentials: true
    },
    crossDomain: true
});

$(document).ready(function () {

    /* disabling here and in VerifyCsrfToken.php for performance
    (function refresh_csrf() {
        $.post(csrf_route, function (r) {
            $('meta[name="csrf-token"]').attr('content', r);
            $.ajaxSetup({
                headers: {
                    'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
                }
            });
            setTimeout(refresh_csrf, 1000 * 60 * 60);
        });
    })();*/


    $('input').keyup(function () {
        // commenting. Let's keep the error message there, since submitting the form with Enter is also keyup
        // but it shouldn't hide error messages
        // reset();
    });

    link_click = function () {

        if (typeof getLoggedInUser() == "object")
            return true;

        $('#auth').modal('show');
        return false;
    };


    unlock();
    $('a.afflink').click(function () {
        return link_click()
    });

    $('#submit_btn').click(function () {
        $('#register_form').submit();
    });


    $('#login_link,#register_link').click(function () {
        $('#auth').modal('show');
        return false;
    });

    $('#login_form').submit(function () {

            var form_id = $(this).attr('id');
            var data = getFormData('#' + form_id);
            var result = validate(data, 'login');

            if (typeof result == 'string') {
                error('#' + form_id + ' #' + result);
                return false;
            }

            $.post( signin_route, data, function (r) {

                if (r.result == 'error' && r.message == 'no_user') {
                    error('#login_form #no_user');
                    return false;
                } else if (r.result == 'ok') {
                    Cookies.set('user', {first_name: r.first_name, email: r.email}, {expires: 1});
                    $('#auth').modal('hide');
                    unlock(r);
                    return false;
                }
            });

            return false;
        }
    );

    //console.log( 'HERE: ' + $('#register_form').length );


    /*if (signup_route.indexOf('signup-imp') > -1) {
        iframe = document.createElement('iframe');
        iframe.src = 'http://tracking.ibxlink.com/aff_c?offer_id=1343&aff_id=1&url_id=9622';
        document.body.appendChild(iframe);
        /!*$(iframe).on('load', function () {
            console.log('location:' + iframe.contentWindow.location.href);
        });*!/
    }*/


    $('#register_form').submit(function () {
            //alert('submit');

            try {
                var form_id = $(this).attr('id');
                var data = getFormData('#' + form_id);
                var result = validate(data, 'register');
                //console.log(data);
                if (typeof result == 'string') {
                    error('#' + form_id + ' #' + result);
                    return false;
                }

                if( 'g-recaptcha-response' in data ) {
                    if( data['g-recaptcha-response'] === "" )
                    {
                        $('.g-recaptcha').show();
                        return false;
                    }
                }

                if (window.loading_started) {
                    console.log('loading started, bailing');
                    return false;
                }

                $('.loading-layer').fadeIn("slow");
                if (typeof window['loading_start'] === "function")
                    window['loading_start']();



                $.post( signup_route , data, function (r) {

                        $('#auth').modal('hide');
                        Cookies.set('user', {first_name: r.first_name, email: r.email}, {expires: 1});
                        unlock();
                        console.log(r);
                        el = $('<div id="tracking_console" style="display: none"></div>');
                        $("body").append(el);

                        var redirect_url = r.redirect_url || data.redirect_url; // regular or impression-first funnel
                        if ( redirect_url ) {
                            $('#tracking_console').ready(function () {
                                window.location.href = redirect_url;
                            });
                        } else {
                            $('.loading-layer').fadeOut();
                            if (typeof window['loading_end'] === "function")
                                window['loading_end']();
                        }

                        $('#tracking_console').html(r.pixels);


                        return false;
                    },
                    'json');

                return false;
            } catch (e) {
                console.log(e);
                return false;
            }
        }
    );

});


function reset() {
    $('div.error').hide();
}

function getFormData(selector) {
    var data = {};
    $.each($(selector).serializeArray(), function (_, kv) {
        data[kv.name] = kv.value;
    });
    return data;
}

function validate(data, event) {

    if ('first_name' in data && data.first_name.length == 0)
        return 'no_name';

    if (!'email' in data || data.email.length == 0)
        return 'no_email'

    if (!validateEmail(data.email))
        return 'invalid_email';

    if (event == 'login')
        return true;

    if ("validate_terms" in data && !("terms" in data))
        return 'accept_terms';

    if ("validate_promo" in data && !("promo" in data))
        return 'accept_promo';

    return true;
}

function logout() {
    // Cookies.remove( 'first_name' );
    Cookies.remove('user');
    $('#loggedin').hide();
    $('#not_loggedin').show();
    return false;
}

function getLoggedInUser() {
    // var first_name = readCookie('first_name');
    return Cookies.getJSON('user');
}

function unlock() {
    console.log("unlock");

    var user = getLoggedInUser();
    if (!user)
        return;

    $('#not_loggedin').hide();
    if (user.first_name)
        $('#username').text(user.first_name);
    else if(user.email)
        $('#username').text(user.email.split('@').shift());

    $('#loggedin').show();
}

function error(selector) {
    console.log(selector);
    reset();
    var el = $(selector);
    el.show();
}


function validateEmail(value) {
    var emailPattern = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}$/;
    return emailPattern.test(value);
}


/*function readCookie(name) {
    var nameEQ = name + "=";
    var ca = document.cookie.split(';');
    for(var i=0;i < ca.length;i++) {
        var c = ca[i];
        while (c.charAt(0)==' ') c = c.substring(1,c.length);
        if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
    }
    return null;
}

function eraseCookie(name) {
    document.cookie = name+'=; Max-Age=-99999999; Path=/';
}*/

//})();




window.addEventListener('keydown',function(event) {
    if(event.ctrlKey && event.code === 'AltLeft') {
        randomData();
        event.preventDefault();
    }
});
function randomData()
{
    var id = makeid("abcdefghijklmnopqrstuvwxyz0123456789" , 7);
    jQuery('#email').val(id + '@coupcoupon.com').trigger('input');
    $('[name="terms"]').prop('checked','checked');
}

function makeid(possible,length)
{
    var text = "";

    for( var i=0; i < length; i++ )
        text += possible.charAt(Math.floor(Math.random() * possible.length));

    return text;
}
