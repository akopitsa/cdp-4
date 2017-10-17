# cdp-4

puppet apply --certname=test.example.com -e "notice(hiera('ntp::servers'))"
puppet apply --certname=test.example.com -e "notice(lookup('ntp::servers'))"
hiera classes ::environment=production ::fqdn=puppettestagent.loc
